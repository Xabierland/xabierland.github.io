---
title: "Gestión de secretos en Kubernetes: Vault, External Secrets Operator y el fin de los secrets estáticos"
author: Xabierland
description: >-
    Los Secrets de Kubernetes son codificación en base64, no cifrado, y el repositorio Git nunca fue sitio para credenciales. La gestión moderna de secretos pasa por motores dinámicos, sincronización declarativa y, sobre todo, por eliminar la existencia misma del secreto estático siempre que sea posible.
date: 2026-08-26 10:00
categories: [DevSecOps, Seguridad de Aplicaciones]
tags: [Vault, External Secrets, Sealed Secrets, Secret Management, Kubernetes, HashiCorp]
---

Pocos problemas tienen tantas soluciones parciales conviviendo en producción como la gestión de secretos. En cualquier cluster maduro coexisten Secrets nativos de Kubernetes, ficheros cifrados en Git, credenciales en variables de entorno de imágenes, contraseñas en *config maps* "temporalmente", un Vault instalado hace dos años que pocos entienden y una nube entera de credenciales en IAM. Este artículo forma parte de la serie [La evolución de la seguridad en aplicaciones modernas](/posts/La-evolucion-de-la-seguridad-en-aplicaciones-modernas/).

## Por qué los Secrets nativos no son suficientes

El objeto Secret de Kubernetes es una conveniencia organizativa, no un mecanismo de seguridad. Su contenido se almacena codificado en base64, lo que es reversible por cualquiera con acceso al objeto. Por defecto, etcd almacena los Secrets en claro en disco, y solo se cifran en reposo si el operador configura explícitamente un KMS (Key Management Service) provider. El control de acceso a los Secrets depende de RBAC, lo que significa que cualquier *service account* con permiso de lectura en su *namespace* ve el contenido sin fricción.

Esa postura por defecto combinada con la costumbre de *GitOps* ha producido el antipatrón más extendido: comprometer credenciales en repositorios, a veces en claro, a veces cifradas con herramientas que nadie sabe rotar. El problema no es que los Secrets existan, es que son el único mecanismo y acaban absorbiendo responsabilidades para las que no fueron diseñados: rotación, auditoría, emisión dinámica, integración con sistemas externos. Cuando un Secret es el *endpoint* final de un sistema más sofisticado, tiene sentido; cuando es el *source of truth*, el sistema está mal diseñado.

## Vault como plano de control de secretos

HashiCorp Vault, disponible desde 2015, convirtió la gestión de secretos en una disciplina con primitivas propias. Su idea central no es "guardar secretos mejor", es cambiar la pregunta: en lugar de custodiar credenciales estáticas, emitirlas dinámicamente bajo demanda con una vida útil acotada.

### Motores dinámicos

Un motor dinámico de base de datos, por ejemplo, no almacena usuarios y contraseñas permanentes. Cuando una aplicación los pide, Vault crea un usuario temporal en la base, le concede los permisos definidos por un *role*, lo entrega al solicitante con una *lease* (por ejemplo, treinta minutos) y lo elimina automáticamente al vencer. El resultado es que no existen credenciales persistentes que puedan robarse: lo que existe es la capacidad, autenticada, de pedirlas. La misma lógica se aplica a motores para nubes (AWS, Azure, GCP), SSH, RabbitMQ, Kafka y otros.

### PKI como servicio y motor de tránsito

El motor PKI (Public Key Infrastructure) de Vault emite certificados X.509 sobre la marcha contra una CA interna configurada, con control fino sobre el *role* autorizado para pedirlos y los campos admitidos. Esa capacidad convierte a Vault en una fuente natural de certificados para mTLS sin montar una jerarquía aparte. El motor *transit* ofrece cifrado como servicio: la aplicación envía datos y recibe criptograma sin tener nunca la clave, lo que es útil para sistemas que necesitan cifrar columnas sensibles sin custodiar el material de clave.

### Métodos de autenticación

La cuestión recurrente de cómo se autentica la aplicación contra Vault se resuelve con métodos específicos. En Kubernetes, el método `kubernetes` permite que un pod se autentique presentando su token de *service account* proyectado, que Vault valida contra el `TokenReview` API del cluster. En infraestructuras cloud, los métodos basados en IAM (AWS, Azure, GCP) permiten autenticación sin secretos previos. AppRole sigue existiendo para sistemas que no tienen identidad nativa que aportar. Cada uno resuelve el mismo problema de *bootstrap*: entregar la primera credencial sin haberla puesto a mano.

## External Secrets Operator y el patrón declarativo

External Secrets Operator (ESO) responde a una necesidad distinta de Vault: no emitir secretos, sino sincronizarlos desde donde vivan hacia objetos Secret de Kubernetes. Define dos CRD (Custom Resource Definitions) principales. `SecretStore` (o `ClusterSecretStore`) describe el proveedor externo, autenticación y ámbito. `ExternalSecret` describe qué secretos extraer, con qué transformación y hacia qué Secret nativo escribir.

El valor no es solo técnico: es filosófico. Convierte el secreto en un recurso declarativo consumido igual que cualquier *manifest*, pero con el origen desacoplado. El *source of truth* vive en el *backend* (Vault, AWS Secrets Manager, GCP Secret Manager, Azure Key Vault, 1Password, Doppler y una lista larga), y el cluster solo mantiene una proyección local que se reconcilia periódicamente. La rotación en el *backend* se propaga automáticamente, con eventos observables desde la capa Kubernetes.

El detalle operativo relevante es que el Secret final sigue siendo un Secret nativo con todos sus defectos. ESO no cifra el objeto, solo lo pone ahí cuando toca. Para escenarios de alta sensibilidad, el patrón se combina con Secrets Store CSI Driver, que monta el secreto como volumen directamente desde el *backend* sin intermediación de un objeto Secret.

## Sealed Secrets y el encaje con GitOps

Sealed Secrets, desarrollado por Bitnami, resuelve el problema contrario: cómo tener secretos en Git sin comprometerlos. Un controlador en el cluster mantiene una clave asimétrica, expone la pública y descifra los objetos `SealedSecret` que llegan al cluster, convirtiéndolos en Secret nativos. El flujo operativo es cifrar localmente con la clave pública y comprometer el `SealedSecret` resultante al repositorio.

El encaje es pulcro para *GitOps* puro: cualquier desarrollador puede cifrar un secreto sin acceso al cluster, el repositorio refleja el estado deseado completo y la reconciliación funciona igual que para cualquier otro recurso. Sus limitaciones son reales: la clave del controlador vive en el cluster y es por cluster (migrar entre clusters exige re-cifrar), rotar la clave maestra es un proceso delicado, y el modelo sigue produciendo Secrets nativos al final. Sealed Secrets es la respuesta correcta cuando el equipo prioriza *GitOps* estricto sobre emisión dinámica, no la respuesta correcta cuando la exigencia es rotación frecuente o cero secretos persistentes.

## Cifrado en reposo y el KMS

Con o sin ESO o Sealed Secrets, etcd sigue siendo el almacén final. La configuración de *encryption at rest* con KMS providers (AWS KMS, Azure Key Vault, GCP KMS, HashiCorp Vault como KMS) cifra los Secret en el nivel de etcd con una clave de datos envuelta por una clave externa que Kubernetes jamás retiene en memoria por largo tiempo. Activarlo es imprescindible en cualquier cluster que custodie cualquier credencial, y omitirlo convierte una copia de seguridad de etcd en una filtración potencial completa.

> Un cluster sin cifrado de etcd es un cluster cuyas credenciales están en un fichero plano en disco. Los backups también.
{: .prompt-warning }

## Patrones de inyección y la convergencia hacia cero secretos

### Sidecar injector de Vault

El patrón del inyector sidecar de Vault agrega a cada pod anotado un contenedor auxiliar que se autentica contra Vault, obtiene los secretos y los escribe en un volumen efímero compartido. La aplicación consume los ficheros sin tocar la API de Vault. La ventaja es la neutralidad frente al código aplicativo: cualquier carga que lea ficheros puede integrarse sin cambios.

### CSI Driver

Secrets Store CSI Driver monta el secreto como volumen directamente desde el *backend*, con sincronización opcional hacia un Secret nativo. Es el camino adecuado cuando se quiere minimizar la presencia del secreto en memoria del pod y evitar reflejarlo como objeto del cluster.

### Identidad sin secretos

La arquitectura más madura deja de inyectar secretos y apuesta por identidad nativa: el *workload* usa su identidad de cluster o cloud (SPIFFE SVID, IRSA, GCP Workload Identity, Azure Workload Identity) para autenticarse contra el *backend*, que le emite lo que necesite sin que exista nunca un secreto persistente. SPIFFE es la identidad, Vault o el *cloud provider* son la fuente de credenciales de acceso a recursos. Esa combinación elimina la categoría entera "secret almacenado en el pod", lo que cambia la superficie de ataque más que cualquier mecanismo incremental de protección.

## Cuándo cada herramienta

El encaje depende del eje de decisión dominante. Si el equipo vive en *GitOps* estricto y los secretos son pocos, estables y específicos por cluster, Sealed Secrets resuelve con fricción mínima. Si hay multitud de cargas consumiendo secretos de diferentes *backends* externos y la prioridad es sincronización declarativa, ESO es la pieza natural. Si el requisito incluye emisión dinámica, rotación agresiva, motores PKI internos o cifrado como servicio, Vault deja de ser opcional. Y si la arquitectura pretende eliminar los secretos estáticos, la pieza fundamental es la identidad *workload* (SPIFFE, Workload Identity cloud) complementada con el motor dinámico.

> El progreso en gestión de secretos se mide mejor por cuántos tipos de secretos estáticos se han eliminado que por cuántos se han cifrado mejor.
{: .prompt-tip }

### Modos de fallo recurrentes

Los incidentes clásicos son consistentes. La ausencia de plan de *disaster recovery* del propio Vault es el más grave: si el *unseal key* se pierde, el Vault queda inservible y con él todas las credenciales que emitía. El registro de auditoría desactivado o no exportado convierte cualquier investigación *post-mortem* en arqueología. La rotación de secretos sin análisis de *blast radius* (qué aplicaciones los usan, cómo reaccionan al cambio, si hay caché) produce caídas en cascada. Y la confusión entre "hemos rotado la credencial" y "hemos invalidado todos los tokens derivados" es el error que convierte una rotación en un falso éxito.

## Encaje arquitectónico

La gestión de secretos es una capa intermedia que conecta identidad (SPIFFE, Workload Identity), criptografía (PKI, KMS) y aplicación. Cuando esas capas están bien separadas, los secretos dejan de ser el problema central y pasan a ser un detalle de implementación: el *workload* tiene identidad, el *backend* emite lo que corresponda y la aplicación consume sin saber dónde vive la clave real. El objetivo razonable no es gestionar mejor los secretos, es reducir progresivamente cuántos existen, y ese camino empieza por diagnosticar sin autoengaño qué se está cifrando y qué se debería dejar de crear.

## Artículos relacionados en esta serie

- [PKI](/posts/PKI/)
- [SPIFFE y SPIRE](/posts/SPIFFE-SPIRE/)
- [Identidad de workload en la nube](/posts/Cloud-Workload-Identity/)
- [mTLS](/posts/mTLS/)
- [Zero Trust](/posts/Zero-Trust/)

## Referencias

- HashiCorp (2015-actualidad). *Vault Documentation*.
- External Secrets Operator. *Project Documentation*.
- Bitnami. *Sealed Secrets: Repository and Specification*.
- Kubernetes. *Documentation: Encrypting Secret Data at Rest*.
- Secrets Store CSI Driver. *Project Documentation*.
- OWASP (2024). *Secrets Management Cheat Sheet*.
- NIST SP 800-57 (2020). *Recommendation for Key Management*.
