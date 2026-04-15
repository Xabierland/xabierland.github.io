---
title: "SPIFFE y SPIRE: identidad criptográfica para cargas de trabajo en entornos dinámicos"
author: Xabierland
description: >-
    En un cluster donde los pods viven minutos y las IPs se reciclan en segundos, la identidad no puede depender ni de la red ni de secretos estáticos inyectados al arrancar. SPIFFE define cómo nombrar cargas de trabajo de forma verificable y SPIRE implementa cómo emitir esa identidad en tiempo de ejecución. Entender ambos es ver el engranaje que conecta PKI, mTLS y Zero Trust.
date: 2026-07-08 10:00
categories: [DevSecOps, Seguridad de Aplicaciones]
tags: [SPIFFE, SPIRE, SVID, Zero Trust, CNCF, Workload Identity, Kubernetes]
---

La identidad de una carga de trabajo moderna es un problema incómodo. Los mecanismos que funcionaron durante décadas (direcciones IP de confianza, secretos inyectados al arrancar, cuentas de servicio estáticas) han envejecido mal en un mundo donde un pod puede nacer, migrar y morir en el intervalo entre dos métricas. SPIFFE (Secure Production Identity Framework For Everyone) nació para dar un nombre verificable a cada *workload* y SPIRE (SPIFFE Runtime Environment) para emitirlo sin depender de secretos previos. Este artículo forma parte de la serie [La evolución de la seguridad en aplicaciones modernas](/posts/La-evolucion-de-la-seguridad-en-aplicaciones-modernas/).

## Por qué hace falta una nueva respuesta

El primer reflejo cuando aparece un servicio nuevo es darle credenciales: un usuario, una clave, un certificado, un token. Ese reflejo funciona mal en cuanto los servicios son miles y efímeros. La entrega inicial de esas credenciales es el eslabón débil: alguien tiene que ponerlas en el pod antes de que el pod se autentique, y ese alguien necesita a su vez credenciales para hacerlo. El problema recursivo del *bootstrap* es lo que en la práctica produce secretos largos almacenados en imágenes, en *config maps* o en repositorios.

La IP tampoco sirve como identidad. En un cluster Kubernetes la misma IP puede pertenecer a servicios distintos en cuestión de segundos, las NAT intermedias borran la distinción y cualquier componente malicioso con acceso a red suplanta a otro sin esfuerzo. Las cuentas de servicio del sistema operativo, igualmente, describen al proceso pero no al *workload* lógico: dos servicios diferentes pueden correr bajo el mismo UID y el mismo contenedor puede servir a dos despliegues distintos.

SPIFFE propone una respuesta distinta: la identidad es un nombre estable, con semántica bien definida, emitido dinámicamente por una autoridad que *atestigua* lo que el *workload* es en el momento de pedirla. El proyecto alcanzó el estado de *Graduated* en la CNCF (Cloud Native Computing Foundation) en 2022, lo que en términos prácticos significa madurez auditada y ecosistema estable.

## Mecánica de SPIFFE

### SPIFFE ID, el nombre verificable

Un SPIFFE ID es un URI con la forma `spiffe://trust-domain/path`. El *trust domain* define el ámbito de autoridad: todos los *workloads* emitidos bajo ese dominio confían en la misma autoridad. El *path* describe qué es la carga, con la granularidad que el diseñador decida (`spiffe://prod.acme.com/ns/billing/sa/invoicing-service`, por ejemplo). La elección de esa estructura no es trivial: demasiado granular y se convierte en un problema de gobernanza, demasiado plana y pierde la capacidad de expresar políticas.

### SVID, la credencial

El SVID (SPIFFE Verifiable Identity Document) es la materialización del SPIFFE ID en una credencial utilizable. Viene en dos formatos: X.509-SVID, un certificado con el SPIFFE ID en el SAN URI (Subject Alternative Name), directamente consumible por cualquier biblioteca TLS estándar; y JWT-SVID, un token firmado en formato JWT para casos donde el transporte no puede negociar TLS (HTTP intermediado por *proxies* que terminan TLS, sistemas de mensajería). El X.509-SVID es la opción natural para mTLS entre servicios y el JWT-SVID para propagación de identidad a través de capas que no pueden sostener una conexión mutua.

### Trust Domain y Federación

El *trust domain* es a la vez un ámbito de confianza y un ámbito de emisión. Todos los *workloads* dentro de un *trust domain* confían en la misma autoridad y reciben SVID firmados por ella. Cuando dos sistemas distintos (dos clusters, dos regiones, dos organizaciones) necesitan hablar entre sí, SPIFFE Federation define cómo intercambiar los *bundles* de confianza para que los *workloads* de un dominio puedan validar SVID emitidos por el otro. La elegancia del diseño está en que la federación es explícita y acotada: se confía en un dominio concreto, no en una autoridad universal.

## Cómo SPIRE emite la identidad

### Server y Agent

SPIRE distribuye la emisión en dos componentes. El SPIRE Server vive centralizado en el *trust domain*, custodia la clave de firma, mantiene el registro de entradas (qué SPIFFE ID corresponde a qué carga y bajo qué condiciones) y firma los SVID. El SPIRE Agent corre en cada nodo, atestigua tanto al nodo como a las cargas locales, y sirve los SVID a los *workloads* a través del Workload API.

La separación tiene consecuencias operativas concretas. El Server es el punto sensible que hay que proteger; el Agent es un componente de cada nodo que solo necesita confianza en su propia atestación. La clave privada de un *workload* se genera en el Agent, la firma la aporta el Server, y el certificado vuelve al Agent para entregarse a la carga. La clave de firma de la CA de SPIRE nunca sale del Server.

### Atestación: la pregunta fundamental

Atestación es el proceso por el cual SPIRE verifica qué es realmente una entidad antes de emitirle una identidad. Se hace en dos niveles. La *node attestation* verifica qué máquina es el Agent: mecanismos como el TPM, la instancia metadata de AWS (IID, Instance Identity Document), la identidad administrada de Azure (MSI) o el token de instancia de GCP (IIT) permiten al Server comprobar criptográficamente que el Agent corre en una máquina concreta con una identidad cloud concreta. La *workload attestation* verifica qué carga está pidiendo una identidad: el Agent inspecciona el proceso solicitante en su namespace local y lo caracteriza por selectores (UID Unix, *labels* de Kubernetes, *service account*, imagen de contenedor, firma de la imagen).

El punto sutil es que SPIRE no pide al *workload* que demuestre quién es mediante un secreto previo: observa lo que ya es visible desde el sistema. Esa diferencia es lo que elimina el problema del *bootstrap*. El *workload* no necesita ninguna credencial inicial porque su identidad se deriva de propiedades observables del entorno, firmadas por cadenas de confianza que empiezan fuera del cluster.

### Rotación automática y vida corta

Los SVID emitidos por SPIRE viven típicamente una hora o menos. El Agent monitoriza la expiración y los rota antes del vencimiento sin intervención del *workload*. Esa vida corta cambia por completo la economía del compromiso: un SVID filtrado tiene una ventana de utilidad acotada y no hay revocación porque la expiración la sustituye. Los sistemas que construyen políticas asumiendo certificados de un año son conceptualmente diferentes de los que asumen rotación cada hora.

### Workload API y el final de los secretos en disco

El Workload API es el contrato estándar por el que una carga solicita su SVID. La comunicación va por un socket Unix expuesto por el Agent local, con atestación del proceso basándose en sus atributos del *kernel*. El SVID nunca se escribe a disco, se entrega en memoria, y se renueva en el mismo canal antes de expirar. Librerías como `go-spiffe` o integraciones en Envoy (a través de SDS, Secret Discovery Service) convierten ese flujo en un detalle transparente para la aplicación.

## Integración con el ecosistema

### Service mesh

Istio puede consumir identidades SPIFFE directamente o integrarse con SPIRE para delegar la emisión. Envoy obtiene los certificados por SDS y los usa en mTLS sin que la aplicación lo note. La implicación es que un *trust domain* SPIFFE puede abarcar la malla y servicios fuera de la malla con la misma identidad, eliminando la discontinuidad típica entre "lo que está en el mesh" y "lo que no".

### Puente con identidades cloud

SPIRE no compite con IRSA, GCP Workload Identity o Azure Workload Identity; puede actuar de puente. Un *workload* obtiene su SVID de SPIRE, lo intercambia en un proveedor cloud por un token nativo y llama a servicios cloud con esa identidad. Ese patrón permite tener una identidad canónica portable entre nubes y traducirla al vocabulario local cuando hace falta.

### SPIFFE no autoriza

La confusión recurrente es pensar que tener un SPIFFE ID ya implica permisos. SPIFFE responde a "quién eres", no a "qué puedes hacer". La decisión de autorización sigue correspondiendo a un motor de política que reciba el SPIFFE ID como entrada: OPA evaluando reglas sobre el ID, OpenFGA consultando relaciones, una *authorization policy* de la malla filtrando por *principal*. El diseño limpio separa emisión de identidad y decisión de acceso, y SPIFFE ocupa solo el primer lado.

> El SPIFFE ID es un sujeto, no un permiso. Quien lo trata como permiso reinventa los antipatrones que justifican separar autenticación y autorización.
{: .prompt-warning }

## Cuándo tiene sentido adoptarlo

SPIRE aporta valor claro cuando el sistema cruza varios entornos (multi-cluster, multi-cloud, on-prem), cuando hay requisitos de identidad criptográfica para cargas que no son HTTP (colas, bases de datos, sistemas legacy integrados con *sidecars*), cuando la rotación automática y la eliminación de secretos estáticos son objetivos explícitos, o cuando la arquitectura quiere estandarizar identidad independientemente de la plataforma subyacente.

No compensa introducirlo en un solo cluster pequeño donde la identidad de *service account* y un poco de IAM de cloud resuelven todos los casos. SPIRE es una pieza de plataforma con un coste operativo propio (alta disponibilidad del Server, gestión del *trust bundle*, monitorización de atestaciones fallidas) que hay que estar dispuesto a pagar.

> Si el equipo aún no tiene claro cómo observa expiraciones de certificados, no es el momento de introducir SPIRE. La identidad efímera magnifica cualquier deuda operativa existente.
{: .prompt-tip }

## Encaje con el resto de la arquitectura

SPIFFE y SPIRE se sitúan entre la PKI (que aporta la criptografía) y mTLS (que aporta el canal), y conviven con cualquier mecanismo de autorización por encima. En una arquitectura Zero Trust coherente, SPIFFE es quien materializa el "nunca confíes, siempre verifica" aplicado a cargas de trabajo: cada conexión presenta un SVID, cada SVID se atestigua en el momento de emisión y cada decisión posterior se toma sobre ese sujeto verificable. Esa continuidad desde el *bootstrap* hasta la decisión de acceso es lo que distingue una implementación Zero Trust real de una colección de controles inconexos.

## Artículos relacionados en esta serie

- [PKI](/posts/PKI/)
- [mTLS](/posts/mTLS/)
- [Identidad de workload en la nube](/posts/Cloud-Workload-Identity/)
- [Gestión de secretos](/posts/Secret-Management/)
- [Zero Trust](/posts/Zero-Trust/)

## Referencias

- SPIFFE. *SPIFFE Specification 1.0*.
- SPIFFE. *X.509-SVID Specification*.
- SPIFFE. *JWT-SVID Specification*.
- SPIFFE. *Workload API Specification*.
- SPIFFE. *Federation Specification*.
- CNCF (2022). *SPIFFE Graduation Announcement*.
- SPIRE. *SPIRE Architecture Documentation*.
