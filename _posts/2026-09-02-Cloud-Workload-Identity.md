---
title: "Identidad de workload en la nube: IRSA, Workload Identity y los SPIFFE de los hyperscalers"
author: Xabierland
description: >-
    Los hyperscalers resolvieron el problema de identidad de workload antes de que SPIFFE fuera mainstream, cada uno a su manera pero con la misma receta: federación OIDC entre el cluster y el IAM cloud. Entender esa mecánica común es ver por qué IRSA, Workload Identity y Azure Workload Identity son, conceptualmente, tres dialectos de la misma idea.
date: 2026-09-02 10:00
categories: [DevSecOps, Seguridad de Aplicaciones]
tags: [IRSA, Workload Identity, Pod Identity, EKS, GKE, AKS, Federación, OIDC]
---

Cada cluster gestionado en un *hyperscaler* ofrece, tarde o temprano, una respuesta a la misma pregunta: cómo un pod llama a una API cloud sin llevar dentro una *access key* estática. Las soluciones tienen nombres distintos en cada nube (IRSA en AWS, Workload Identity en GCP, Workload Identity en Azure) pero el mecanismo subyacente es el mismo: federación OIDC (OpenID Connect) entre el emisor de tokens del cluster y el IAM (Identity and Access Management) del proveedor. Este artículo forma parte de la serie [La evolución de la seguridad en aplicaciones modernas](/posts/La-evolucion-de-la-seguridad-en-aplicaciones-modernas/).

## Por qué la federación OIDC resolvió el problema

La historia precedente fue una sucesión de malas respuestas. Durante años los pods cargaban *access keys* estáticas como variables de entorno, con la doble desgracia de vivir para siempre y quedar expuestas en cualquier *core dump* o log mal configurado. Los roles de nodo (asignar a la máquina EC2 un rol IAM que todos los pods heredaban) colapsaban el principio de mínimo privilegio: todos los pods en el nodo tenían los mismos permisos, los más sensibles dictados por el más laxo. La primera iteración de soluciones por pod, como kube2iam o kiam, interceptaba el metadata service con proxies y sufría problemas de carrera, latencia y compromiso.

La respuesta moderna cambia de paradigma. Kubernetes ya sabe emitir tokens firmados para cada ServiceAccount. Si el cluster expone un *issuer* OIDC público con la clave pública de firma, cualquier sistema que confíe en ese *issuer* puede validar los tokens emitidos por el cluster. El IAM cloud se convierte en ese sistema: acepta un token del cluster como prueba de identidad, verifica que el *subject* coincide con la política definida y responde con credenciales cloud de corta vida. Los secretos estáticos desaparecen y la identidad del pod pasa a derivarse de su ServiceAccount y del cluster en el que corre.

## Mecánica común

### El cluster como proveedor OIDC

Todo cluster Kubernetes moderno puede actuar como proveedor OIDC. El kube-apiserver emite tokens para las ServiceAccount a petición del TokenRequest API, firmados con una clave del cluster y con *claims* estándar (`iss`, `sub`, `aud`, `exp`). Si esos *claims* son accesibles públicamente en los dos *endpoints* habituales (`/.well-known/openid-configuration` y `/openid/v1/jwks`), cualquier *relying party* externo puede validar la firma.

La versión usable de este mecanismo son los *projected service account tokens*, con audiencia restringida (`audience`) y duración acotada (desde 2022 con *bound token* por defecto, y con expiración máxima que en 2024 pasó a un año en varias distribuciones). La restricción de audiencia es crítica: un token válido para el IAM cloud no debe poder usarse contra la API del cluster ni viceversa, y cada lado solo acepta tokens emitidos para su audiencia concreta.

### El IAM cloud como *relying party*

La segunda mitad del mecanismo es que el IAM cloud confía en el *issuer* del cluster. La configuración típica incluye registrar la URL del *issuer*, descargar la JWKS (JSON Web Key Set) con las claves públicas de firma y definir una política que enlace un *subject* concreto (típicamente la ServiceAccount específica de un *namespace*) con un rol o identidad cloud. Cuando el pod presenta su token, el IAM valida firma, *issuer*, *audience*, `exp` y `sub`, y a cambio entrega credenciales cloud de corta vida.

La traducción concreta varía entre proveedores, pero el patrón es siempre el mismo: token del cluster a cambio de credenciales cloud.

## Las tres implementaciones

### AWS: IRSA y EKS Pod Identity

IRSA (IAM Roles for Service Accounts) fue la primera implementación masiva en 2019. El mecanismo usa STS (Security Token Service) y la operación `AssumeRoleWithWebIdentity`: el pod obtiene su token de ServiceAccount, lo envía a STS junto con el ARN del rol IAM, y recibe credenciales temporales del rol. La configuración del lado IAM es una *trust policy* que acepta tokens de un *issuer* OIDC concreto cuando el `sub` tiene la forma `system:serviceaccount:namespace:serviceaccount-name`.

EKS Pod Identity, introducido en 2023, resuelve los mismos casos de uso con otra mecánica: un agente en cada nodo expone un *endpoint* local que el SDK cloud consulta, y las credenciales fluyen sin federación OIDC explícita. La ventaja es operativa: se evita gestionar el *issuer* público, se simplifica la configuración en *trust policies* y el *blast radius* de una clave de firma comprometida se acota. IRSA sigue vigente y es más interoperable (cualquier cluster con OIDC puede usarlo, incluidos clusters fuera de AWS); Pod Identity es específico de EKS y más sencillo.

### GCP: Workload Identity

GCP Workload Identity para GKE establece un enlace entre una ServiceAccount de Kubernetes (KSA) y una ServiceAccount de Google (GSA). La KSA se anota con la identidad GSA correspondiente y la GSA concede a la KSA el rol `iam.workloadIdentityUser`. El pod obtiene un token *projected*, el SDK de Google lo intercambia en STS por un token federado y ese token se usa para acceder a APIs cloud con los permisos de la GSA.

La evolución reciente tiende a eliminar el paso intermedio de la GSA con *direct resource access* en algunos servicios, donde las políticas IAM se asocian directamente a la KSA vía *principal identifier*. La dirección clara es reducir la indirección.

### Azure: Workload Identity

Azure Workload Identity, que sustituyó a Pod Identity v1 (cuyo soporte finalizó en 2024), sigue exactamente el patrón OIDC. AKS expone un *issuer* OIDC, el *workload* obtiene su token proyectado, y Entra ID (anteriormente Azure AD) lo acepta como credencial federada de una identidad administrada o de una *app registration*. La configuración en Entra ID registra el *issuer* del cluster y el *subject* esperado, y concede los permisos Azure correspondientes a la identidad.

La transición desde Pod Identity v1, basado en un componente que interceptaba llamadas al metadata service, a Workload Identity, basado en OIDC estándar, es el ejemplo canónico de cómo los mecanismos específicos de cada nube han convergido hacia el mismo patrón federado.

## Federación más allá del cluster local

Lo elegante del modelo es que nada exige que el cluster esté en la misma nube que el IAM que emite credenciales. GCP Workload Identity Federation, AWS roles con IdP OIDC externos y Azure Workload Identity con *issuer* externo permiten que un cluster on-prem, o un cluster en otra nube, consuma APIs cloud sin usar claves estáticas. El mismo cluster Kubernetes de un *data center* propio puede llamar a S3, Cloud Storage y Blob Storage autenticándose por federación contra cada IAM.

Esa portabilidad es el argumento más fuerte contra la tentación de guardar claves estáticas "por comodidad": el coste de configurar la federación es inicial y la ganancia (ausencia de secretos, rotación automática, auditoría nativa del cloud) es permanente.

## Paralelismo conceptual con SPIFFE

El patrón recuerda inmediatamente a SPIFFE. La ServiceAccount es el *subject*, el token proyectado es el SVID, el cluster es la autoridad de emisión y el IAM cloud es el *relying party* que valida. Las diferencias son de ámbito y gobernanza. Las soluciones de los *hyperscalers* son gestionadas, integradas con sus IAM respectivos y cómodas cuando la arquitectura vive dentro de una sola nube. SPIFFE es portable, neutro entre nubes y consistente en entornos híbridos, a costa de montar y operar SPIRE.

La elección no es excluyente. Es habitual usar identidad *cloud* para acceso a APIs del proveedor y SPIFFE para identidad este-oeste entre servicios en mTLS, con Vault o un motor equivalente intermediando para traducir identidades cuando se necesita.

> Elegir entre Workload Identity del *cloud* y SPIFFE no es una decisión técnica pura, es una decisión de gobernanza. Si la arquitectura pretende ser portable, SPIFFE paga su coste; si vive en una nube, la solución nativa gana en simplicidad.
{: .prompt-info }

## Modos de fallo habituales

### Validación incompleta en el IAM

El error más repetido es definir la *trust policy* sin comprobar `sub` y `aud` con suficiente especificidad. Una política que valida solo el *issuer* acepta cualquier ServiceAccount del cluster, convirtiendo el *trust boundary* en el cluster entero en lugar de en una carga concreta. La regla operativa es pinear `sub` al *namespace* y nombre exactos, y exigir `aud` específica.

### Permisos excesivos en el mapeo

El mecanismo entrega credenciales del rol IAM enlazado. Si ese rol tiene permisos amplios, cada pod con acceso a la ServiceAccount los hereda. La granularidad correcta asigna roles por función concreta, no por equipo o aplicación. Un rol "del servicio de facturación" que pueda leer cualquier *bucket* es el mismo antipatrón que los roles de nodo compartidos, solo que disfrazado de moderno.

### Expiración del token y clientes con caché

Los tokens proyectados tienen expiración. Los SDK cloud modernos renuevan automáticamente, pero cualquier capa propia que lea el token una vez y lo cachee indefinidamente rompe al llegar el vencimiento. Los incidentes clásicos aparecen después del despliegue inicial, cuando la carga pasa de horas a días corriendo y los tokens cacheados empiezan a fallar.

### El issuer no público

El cluster tiene que exponer su *issuer* OIDC públicamente (o, al menos, accesible para el IAM cloud). Distribuciones on-prem que cierran el *endpoint* de *issuer* por error rompen la federación sin aviso claro, y el diagnóstico es confuso porque los tokens se emiten correctamente pero el IAM no puede validarlos.

> Una *trust policy* que no pinea `sub` es equivalente a conceder ese rol al cluster entero. El aviso del propio proveedor rara vez es suficientemente estridente.
{: .prompt-warning }

## Cuándo es la pieza correcta

La identidad *workload* cloud es la respuesta adecuada siempre que el *workload* vive en un *cluster* gestionado, consume APIs del mismo proveedor y el equipo ya opera con ese IAM. Su coste de adopción es bajo, su ganancia es inmediata y la alternativa (secretos estáticos) está fuera de toda discusión en contextos mínimamente serios.

Cuando la arquitectura cruza nubes, integra sistemas on-prem y cargas legacy, o quiere una capa de identidad común independiente del proveedor, conviene complementar con SPIFFE o al menos con un *broker* de identidad (típicamente Vault) que traduzca entre dialectos. El error sería elegir una sola solución como respuesta universal: cada una resuelve un ámbito y el diseño maduro las combina.

## Encaje con el resto de la arquitectura

Workload identity cloud es la capa de autenticación entre el *workload* y el plano cloud. Por debajo hay PKI (firma de tokens, *issuer* OIDC) y por encima hay autorización (políticas IAM específicas). Junto a ella, SPIFFE resuelve la identidad este-oeste entre servicios y Vault o equivalentes gestionan la emisión de credenciales a sistemas que no son APIs cloud nativas. Zero Trust se apoya en todas estas piezas a la vez: cada llamada se autentica, cada identidad se deriva de atributos verificables y ningún secreto estático atraviesa el sistema. El objetivo común es que la palabra "credencial" deje de describir un fichero y pase a describir una capacidad autenticada, efímera y auditable.

## Artículos relacionados en esta serie

- [SPIFFE y SPIRE](/posts/SPIFFE-SPIRE/)
- [Gestión de secretos](/posts/Secret-Management/)
- [PKI](/posts/PKI/)
- [Zero Trust](/posts/Zero-Trust/)
- [OpenID Connect](/posts/OIDC/)

## Referencias

- RFC 8693 (enero 2020). *OAuth 2.0 Token Exchange*.
- OpenID Foundation (2014). *OpenID Connect Core 1.0*.
- Kubernetes. *KEP-1205: Bound Service Account Tokens*.
- AWS (2019). *IAM Roles for Service Accounts (IRSA)*.
- AWS (2023). *EKS Pod Identity*.
- GCP. *Workload Identity for GKE*.
- GCP. *Workload Identity Federation*.
- Azure (2024). *Workload Identity y guía de migración desde Pod Identity v1*.
