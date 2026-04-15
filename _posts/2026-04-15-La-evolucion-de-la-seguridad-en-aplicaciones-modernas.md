---
title: La evolución de la seguridad en aplicaciones modernas
author: Xabierland
description: >-
    Un recorrido histórico por los conceptos, protocolos y patrones que definen la seguridad de las aplicaciones distribuidas actuales. Desde los primeros sistemas cerrados de los años 80 hasta las arquitecturas cloud-native basadas en Kubernetes, cada pieza del puzle se introdujo para resolver un problema concreto de su tiempo. Este artículo es el mapa que conecta todas esas piezas y sirve como punto de entrada a la serie completa.
date: 2026-04-15 10:00
categories: [DevSecOps, Seguridad de Aplicaciones]
tags: [Seguridad, Arquitectura, Microservicios, Kubernetes, Historia, OAuth2, JWT, OIDC, mTLS, SPIFFE, PKI, RBAC, ABAC, ReBAC, API Gateway, Service Mesh, IdP, Zero Trust]
pin: true
---

La seguridad de las aplicaciones no se diseñó de una vez. Cada concepto que hoy aceptamos como estándar (OAuth, JWT, mTLS, SPIFFE, OPA) nació como respuesta a un problema concreto que alguien no podía resolver con las herramientas que tenía en ese momento. Entender el orden en el que aparecieron esos problemas es la mejor forma de entender por qué las arquitecturas modernas tienen la forma que tienen, y por qué simplificarlas en exceso acaba reintroduciendo vulnerabilidades que la industria ya resolvió hace veinte años.

Este artículo recorre ese hilo histórico desde los primeros sistemas multiusuario hasta las arquitecturas cloud-native actuales. Cada sección introduce un concepto en el momento en que apareció y enlaza al artículo completo de la serie donde se profundiza. La intención no es enseñar seguridad desde cero, sino conectar piezas que habitualmente se estudian por separado y revelar la lógica evolutiva que las une.

## Los orígenes: identidad en sistemas cerrados

En los mainframes de los años 70 y 80 la seguridad era un problema resuelto por el sistema operativo. El usuario tenía una cuenta local, una contraseña almacenada en el mismo host y un conjunto de permisos asociados a esa cuenta. Autenticación y autorización vivían en el mismo registro. Este modelo funcionaba porque el universo de usuarios era cerrado, administrado por una única autoridad y rara vez se comunicaba con sistemas externos.

Cuando las redes universitarias empezaron a necesitar autenticación entre hosts distintos, el Project Athena del MIT publicó en 1988 **Kerberos**, el primer protocolo serio de autenticación distribuida basado en tickets criptográficos y en un tercero de confianza (el KDC). Kerberos introdujo ideas que todavía se usan: el ticket como artefacto de autenticación portable, la separación entre autenticación inicial y autorización por servicio, y la delegación controlada. Microsoft lo adoptó en Windows 2000 como base de Active Directory, y todavía hoy miles de millones de autenticaciones diarias se apoyan en él.

> Kerberos no desapareció. Sigue autenticando a millones de empleados en empresas que tienen Active Directory como columna vertebral. Entenderlo es imprescindible para cualquier arquitectura que necesite federar con esos entornos.
{: .prompt-info }

Los detalles del protocolo, sus limitaciones (sincronización horaria, dependencia de red al KDC) y su papel actual en la federación con IdPs modernos están en el artículo dedicado a [Kerberos](/posts/Kerberos/).

Paralelamente a Kerberos se consolidó **LDAP (Lightweight Directory Access Protocol, RFC 4511, 2006)** como almacén de identidad empresarial, y **Active Directory** (Microsoft, 2000) como su implementación dominante combinando LDAP, Kerberos y DNS en un único producto. Durante dos décadas el directorio corporativo fue la fuente canónica de usuarios, grupos y políticas. Entender LDAP/AD es imprescindible para cualquier arquitectura que necesite integrarse con entornos empresariales reales, donde los IdPs modernos se sincronizan con el directorio en lugar de reemplazarlo. El artículo sobre [LDAP y Active Directory](/posts/LDAP-Active-Directory/) desarrolla el modelo de directorio, la jerarquía DIT, los bind modes y el papel actual como fuente de identidad tras los IdPs.

También conviene dejar clara desde el principio la distinción que más confusión arrastra: **autenticación** (quién eres) no es **autorización** (qué puedes hacer). Las dos responden a preguntas distintas, viven en capas distintas y, cuando se mezclan en el código de negocio, producen arquitecturas imposibles de auditar. El artículo sobre [autenticación y autorización](/posts/Autenticacion-y-Autorizacion/) desarrolla este principio y sus implicaciones.

## La web y la ilusión de las sesiones

Los años 90 trajeron HTTP, un protocolo deliberadamente sin estado. Las aplicaciones web necesitaban recordar quién estaba conectado entre peticiones, y la solución fue la **sesión de servidor**: un identificador enviado en una cookie, asociado a un objeto en memoria del servidor con los datos del usuario. Funcionaba bien en el modelo monolítico, mal en sistemas distribuidos y pésimamente cuando varios servidores tenían que compartir ese estado.

La autenticación se hacía con **Basic Auth** (la contraseña en base64 en cada petición, en texto plano si no había TLS) o **Digest Auth** (un hash con nonce, más seguro pero poco usado). Ambos mecanismos forzaban a la aplicación a almacenar directamente las contraseñas o sus hashes, y atarse a ellas como único método de verificación. No existía delegación, no existía federación: cada aplicación era una isla con su propio sistema de credenciales.

## La federación empresarial: SAML

A principios de los 2000 las empresas empezaron a necesitar que sus empleados accedieran con el mismo login a decenas de aplicaciones internas y a servicios SaaS externos. Replicar usuarios en cada sistema era inaceptable por coste, seguridad y cumplimiento. Nace el concepto de **Single Sign-On federado**: la aplicación (el Service Provider) delega la autenticación a un Identity Provider (el IdP) común.

**SAML 2.0** (OASIS, marzo 2005) consolidó esta idea. El IdP emite una *assertion* XML firmada con los datos del usuario, el SP la consume y confía en ella. A pesar de ser verbosa, XML-heavy y haber heredado problemas históricos (ataques de XML Signature Wrapping, confusión de algoritmos), SAML sigue dominando la federación empresarial B2B dos décadas después. El artículo sobre [SAML](/posts/SAML/) explica su modelo, sus *bindings* y por qué OIDC no lo ha podido jubilar.

## La era de las APIs y el problema de la delegación

Alrededor de 2007 las aplicaciones web dejaron de vivir aisladas. Twitter, Facebook, Flickr y otros ofrecían APIs para que aplicaciones de terceros leyeran datos de los usuarios, y la única forma de hacerlo era pedir al usuario su contraseña para que la aplicación la guardara. Este modelo tenía tres problemas gravísimos: la aplicación quedaba con la contraseña real, no había forma de revocar acceso sin cambiar esa contraseña, y no existía granularidad (si te dejaba leer el perfil también te dejaba borrar la cuenta).

La respuesta fue **OAuth 1.0** (2010, RFC 5849) y posteriormente **OAuth 2.0** (octubre 2012, RFC 6749). OAuth 2.0 simplificó el modelo apoyándose en HTTPS como capa de transporte segura y definió cuatro roles: Resource Owner, Client, Authorization Server y Resource Server. También introdujo los *grant types* (Authorization Code, Client Credentials, Implicit y Resource Owner Password Credentials) y el concepto de *scope* como unidad de granularidad.

> OAuth 2.0 es un protocolo de **delegación de autorización**, no de autenticación. El 80% de los errores arquitectónicos en este espacio vienen de confundirlos.
{: .prompt-warning }

El artículo sobre [OAuth 2.0](/posts/OAuth2/) desmenuza los roles, los flujos y las decisiones de diseño que han definido el ecosistema actual.

## El token como artefacto portable: JWT

OAuth 2.0 definió el protocolo pero dejó abierto el formato del token. Durante varios años cada implementación usaba lo que quería (tokens opacos con introspección, tokens propietarios, etc.). En mayo de 2015 el IETF publicó **RFC 7519 (JSON Web Token)** estandarizando un formato firmado, autocontenido y verificable sin consultar una base de datos.

JWT tiene tres partes codificadas en Base64url: cabecera, *payload* con los *claims*, firma. El formato firmado (JWS, RFC 7515) es el canónico; el cifrado (JWE, RFC 7516) existe pero se usa menos. La ventaja de JWT es operacional: cualquier servicio con la clave pública puede validar el token sin hablar con el emisor. La desventaja es estructural: una vez emitido, el token es válido hasta su expiración, y la revocación se convierte en un problema sin solución perfecta.

El artículo sobre [JWT](/posts/JWT/) cubre la anatomía del token, las vulnerabilidades clásicas (ataque `alg:none`, confusión RS256/HS256), el problema de la revocación y las estrategias que los equipos usan para aproximarse a él.

### Una alternativa olvidada: Macaroons

JWT no es la única forma de construir un token autocontenido. En 2014 Google publicó **Macaroons: Cookies with Contextual Caveats for Decentralized Authorization in the Cloud**, un formato de credencial que permite a cualquier portador **atenuar** el token añadiendo restricciones (*caveats*) sin necesidad de hablar con el emisor. Una macaroon se construye encadenando HMACs, de modo que cada *caveat* añadido se vuelve criptográficamente inseparable del token, y puede incluir condiciones verificables offline (expiración, scope, IP) o *third-party caveats* que exigen probar una condición ante un tercero.

La idea resuelve elegantemente problemas que JWT deja abiertos (delegación con reducción de privilegios, offline attenuation, capability-based security), pero la falta de estandarización IETF y el dominio del ecosistema OAuth/JWT le impidió llegar a ser mainstream. Donde sí se usan es en sistemas como Google Cloud Storage, Hyperdrive o Tailscale. El artículo sobre [Macaroons](/posts/Macaroons/) explica el modelo de *caveats*, las diferencias con JWT y los escenarios donde siguen siendo la respuesta correcta.

## OIDC: cuando OAuth 2.0 aprendió a decir quién eres

OAuth 2.0 dejó deliberadamente fuera la autenticación, pero la industria seguía necesitándola y empezó a implementarla encima de OAuth de formas creativas e incompatibles entre sí. En febrero de 2014 la OpenID Foundation publicó **OpenID Connect Core 1.0** como una capa de identidad estándar sobre OAuth 2.0.

OIDC añade tres elementos: el **ID Token** (un JWT que prueba que la autenticación ocurrió), el **UserInfo Endpoint** (para obtener claims adicionales del usuario) y el **Discovery Document** (`.well-known/openid-configuration`) que permite a cualquier cliente descubrir los endpoints, algoritmos y claves públicas del IdP. Ese trío convirtió OIDC en el estándar *de facto* de autenticación federada en aplicaciones modernas, desde "Iniciar sesión con Google" hasta integraciones B2B empresariales.

El artículo sobre [OIDC](/posts/OIDC/) explica la diferencia entre Access Token e ID Token, los *response types*, el papel del *nonce* y por qué validar `iss`, `aud`, `exp` y la firma es obligatorio aunque el SDK parezca hacerlo automáticamente.

### El problema operacional: sesiones, refresh tokens y revocación

Un token autocontenido como JWT es fácil de emitir pero difícil de revocar. Cuando un usuario cierra sesión, cambia su contraseña, o se detecta un dispositivo comprometido, el sistema tiene que invalidar inmediatamente el acceso, y un JWT firmado válido hasta su expiración no se deja matar sin infraestructura adicional. La respuesta moderna combina *access tokens* de vida corta (minutos), *refresh tokens* de vida larga rotados en cada uso (RFC 6749, RFC 6819), listas de revocación, *token introspection* (RFC 7662) y *session management* en el IdP (OpenID Connect Session Management, Front-Channel y Back-Channel Logout).

El artículo sobre [gestión de sesiones y revocación](/posts/Sesiones-y-Revocacion/) desarrolla los patrones de ciclo de vida de tokens, la rotación de *refresh tokens*, los modos de logout en OIDC y los compromisos arquitectónicos entre tokens opacos con introspección y tokens autocontenidos con revocación aproximada.

### Señales continuas: CAEP y Shared Signals Framework

Los patrones anteriores revocan cuando alguien lo pide, pero no cuando algo cambia. Si un dispositivo se detecta comprometido, si el riesgo del usuario sube, si una sesión se considera sospechosa a mitad de uso, el IdP y los *relying parties* necesitan comunicarse en tiempo real. La **OpenID Shared Signals Framework** y su perfil **CAEP (Continuous Access Evaluation Profile)**, formalizados por la OpenID Foundation entre 2022 y 2024, definen un bus de eventos asíncronos entre entidades de identidad: *session-revoked*, *token-claims-change*, *credential-change*, *device-compliance-change*. Google (Cross-Account Protection), Microsoft (Continuous Access Evaluation en Entra ID) y Okta han desplegado esta arquitectura.

CAEP es, en la práctica, lo que convierte Zero Trust de un eslogan en un sistema funcional: la "verificación continua" se apoya en esos eventos. El artículo sobre [CAEP y Shared Signals](/posts/CAEP-Shared-Signals/) desarrolla el modelo de *transmitters* y *receivers*, los tipos de eventos, el transporte (SET, JWT de *Security Event Tokens*) y la integración con IdPs modernos.

## El cliente público como eslabón débil: PKCE

El flujo *Authorization Code* de OAuth 2.0 funciona bien cuando el cliente es **confidencial** (puede guardar un *client secret*). Con aplicaciones móviles y SPAs aparece un problema: son **clientes públicos**, no pueden guardar secretos, y el *authorization code* puede ser interceptado (por otra app registrada con el mismo esquema de URL, por ejemplo) para intercambiarlo por un token.

**PKCE (Proof Key for Code Exchange, RFC 7636, septiembre 2015)** resolvió esto con un mecanismo sencillo: el cliente genera un `code_verifier` aleatorio, calcula su hash SHA-256 (`code_challenge`) y lo envía en la petición inicial; al intercambiar el código por el token debe demostrar conocimiento del verifier original. Un atacante que intercepte el código no puede canjearlo sin el verifier.

PKCE empezó siendo recomendación para móviles y acabó siendo obligatorio para todos los clientes en OAuth 2.1. El artículo sobre [PKCE](/posts/PKCE/) detalla el ataque que lo motivó y por qué se ha convertido en la base del OAuth moderno.

## Los proveedores de identidad como infraestructura

A medida que OAuth, OIDC y SAML se consolidaron, construir un IdP propio dejó de tener sentido para la mayoría de los equipos. **Keycloak** (open source, Red Hat) y **Okta** (SaaS desde 2009, con Auth0 en su cartera desde 2021) dominan el mercado, junto con Entra ID de Microsoft y opciones cloud-nativas como Cognito o Google Identity Platform.

Todos los IdP modernos implementan simultáneamente tres papeles: Authorization Server de OAuth 2.0, OpenID Provider de OIDC y SAML IdP. Además gestionan ciclo de vida de usuarios, MFA, políticas de *sign-on*, federación con otros IdPs e integración con directorios corporativos. El artículo sobre [Keycloak y Okta](/posts/Keycloak-Okta-IdP/) compara modelos operacionales, criterios de decisión self-hosted vs SaaS y los conceptos comunes (realm/tenant, client, scope, claim mapper) que cualquier arquitecto debe dominar antes de evaluar un IdP.

### CIAM: la identidad orientada a consumidores

El IAM clásico asume empleados con cuentas creadas por un administrador. Las aplicaciones de consumo invierten ese modelo: millones de usuarios se auto-registran, combinan login social con verificación por email, aceptan términos de privacidad, recuperan contraseñas y esperan ergonomía premium. **CIAM (Customer Identity and Access Management)** es la categoría de producto que responde a ese caso. Auth0 (adquirida por Okta en 2021), Cognito, Firebase Authentication, FusionAuth, Stytch y WorkOS dominan el espacio.

Los retos del CIAM son distintos: progressive profiling, gestión de consentimiento GDPR, prevención de bot registration, account recovery sin soporte humano, escalabilidad a millones de cuentas. El artículo sobre [CIAM](/posts/CIAM/) desarrolla las diferencias con IAM empresarial, los patrones de login social seguros y los criterios de elección entre SaaS y self-hosted.

### OpenID Federation: federación de IdPs a escala

Dos IdPs pueden federarse entre sí mediante confianza explícita (intercambio de metadatos firmados), pero ese modelo no escala cuando la red de confianza incluye docenas o cientos de entidades. **OpenID Federation (RFC 9678, octubre 2024)** resuelve este problema con un modelo de *trust chains*: cada entidad publica un *entity statement* firmado con sus claves, las autoridades intermedias encadenan statements, y cualquier *relying party* puede resolver dinámicamente la confianza hacia un IdP desconocido siguiendo la cadena hasta un *trust anchor* común.

Este patrón es la base técnica de iniciativas como GAIA-X, la eIDAS 2.0 European Digital Identity Wallet y federaciones sectoriales (sanidad, banca, educación). El artículo sobre [OpenID Federation](/posts/OpenID-Federation/) explica el modelo de *entity statements*, la resolución de *trust chains* y los casos de uso que OIDC plano no cubre.

### SCIM: el protocolo de provisioning entre IdP y aplicaciones

Un IdP sabe quiénes son los usuarios, pero cada aplicación downstream necesita su propia copia de esos usuarios y sus grupos. Antes de SCIM cada integración inventaba su propio mecanismo: exportaciones CSV, scripts de sincronización LDAP, APIs propietarias. **SCIM (System for Cross-domain Identity Management, RFC 7643 y RFC 7644, 2015)** estandarizó el modelo y el protocolo: un esquema REST/JSON para representar usuarios y grupos, y operaciones CRUD para crear, actualizar, buscar y desactivar identidades desde el IdP hacia cualquier aplicación compatible.

Hoy es la vía canónica para el *joiner/mover/leaver* empresarial: cuando un empleado se incorpora, cambia de equipo o sale de la empresa, el IdP propaga esos cambios a decenas de SaaS mediante SCIM sin intervención humana. El artículo sobre [SCIM](/posts/SCIM/) explica el esquema, los *endpoints*, los modos *push* y *pull*, y por qué es una pieza crítica en cualquier arquitectura SaaS B2B.

### IGA: la capa de gobernanza sobre IAM

SCIM resuelve cómo propagar identidades. **Identity Governance and Administration (IGA)** resuelve quién debe tenerlas y cómo se auditan periódicamente. La disciplina nació en los 2000 impulsada por SOX y SOC 2, y productos como SailPoint, Saviynt y Omada se especializaron en cuatro procesos: *access requests* (flujos de aprobación), *access reviews* (recertificación periódica por managers), *segregation of duties* (SoD, detección de combinaciones de permisos prohibidas) y *role mining* (descubrimiento de roles a partir de uso real).

En entornos regulados (SOC 2, ISO 27001, SOX, DORA) la ausencia de IGA no es una elección arquitectónica, es un hallazgo de auditoría. El artículo sobre [Identity Governance and Administration](/posts/Identity-Governance-IGA/) desarrolla el modelo de *joiner/mover/leaver*, las *access reviews* y la integración con IdPs y SCIM.

### MFA y el fin de la contraseña como único factor

Durante décadas la contraseña fue el único factor de autenticación que la mayoría de sistemas exigía, y durante décadas fue la principal vía de compromiso. El salto cualitativo lo trajo **MFA (Multi-Factor Authentication)**: combinar dos o más factores independientes de las categorías clásicas (algo que sabes, algo que tienes, algo que eres). **TOTP (RFC 6238, 2011)** y **HOTP (RFC 4226, 2005)** estandarizaron los códigos generados por aplicaciones como Google Authenticator o Authy; los SMS y las notificaciones push añadieron modalidades menos seguras pero masivas.

NIST SP 800-63B categoriza los factores por nivel de garantía (AAL1, AAL2, AAL3). El artículo sobre [MFA](/posts/MFA/) cubre los factores, las vulnerabilidades de SMS y push (SIM swapping, MFA fatigue), los patrones de step-up authentication y el contexto en el que Passkeys los empieza a reemplazar.

### FIDO2, WebAuthn y Passkeys: el fin de la contraseña

MFA sobre contraseña sigue teniendo el talón de Aquiles de la contraseña. **FIDO2** (FIDO Alliance, 2018) y su estándar web **WebAuthn** (W3C Recommendation, 2019) propusieron eliminar la contraseña directamente: el usuario demuestra posesión de una clave privada almacenada en un autenticador (clave física YubiKey, enclave TPM del equipo, Secure Enclave del móvil) mediante un *challenge-response* criptográfico. No hay secreto compartido que robar, no hay phishing reusable, y la clave privada nunca abandona el dispositivo.

**Passkeys** (Apple, Google, Microsoft, 2022) son FIDO2 credentials sincronizadas a través del ecosistema del proveedor (iCloud Keychain, Google Password Manager, Microsoft Authenticator). Su propagación masiva en 2023-2024 marca el inicio de la era *passwordless*. El artículo sobre [FIDO2, WebAuthn y Passkeys](/posts/FIDO2-WebAuthn-Passkeys/) explica el modelo de atestación, los *relying parties*, la diferencia entre passkeys sincronizadas y device-bound, y las implicaciones arquitectónicas de adoptarlas.

## Zero Trust: la filosofía que lo reencuadra todo

En paralelo al avance de los protocolos, en 2010 John Kindervag (entonces en Forrester) acuñó el término **Zero Trust**. Google publicó en 2014 el paper de **BeyondCorp** describiendo cómo, tras el ataque Operation Aurora, había eliminado la idea de red interna confiable. En agosto de 2020 el NIST formalizó el modelo en **SP 800-207**.

Zero Trust no es un producto ni un protocolo: es una filosofía de diseño. Parte de la premisa de que el perímetro es un mito (la red interna es tan hostil como Internet), y propone que cada acceso se verifique en el momento contra identidad, contexto y política. Los siete principios del NIST articulan esa idea. En la práctica, Zero Trust es el marco que da coherencia a piezas aparentemente sueltas: mTLS para la confianza del canal, SPIFFE para la identidad del workload, OPA para la autorización contextual, API Gateway como *Policy Enforcement Point*.

El artículo sobre [Zero Trust](/posts/Zero-Trust/) desarrolla los siete tenets, el modelo PDP/PEP, la diferencia con ZTNA y VPN tradicional, y cómo encaja en Kubernetes.

### Accesos privilegiados, efímeros y auditables: PAM

Zero Trust obliga a replantear un caso particularmente sensible: el acceso administrativo a producción. Un DBA que necesita conectarse a una base de datos, un SRE que debe depurar un nodo, un auditor que debe exportar logs son accesos con privilegios elevados que tradicionalmente se resolvían con cuentas compartidas de larga duración o con sudo sin control. **Privileged Access Management (PAM)** reemplaza ese modelo por accesos efímeros, justificados y auditados: credenciales generadas al momento, válidas minutos u horas, emitidas bajo aprobación y con sesión grabada.

El patrón *just-in-time access* (JIT) y el *break-glass* (acceso de emergencia con auditoría reforzada) se han convertido en estándar en entornos regulados. Herramientas como CyberArk, HashiCorp Boundary, Teleport, StrongDM o AWS IAM Identity Center con sesiones temporales implementan este modelo. El artículo sobre [Privileged Access Management](/posts/Privileged-Access-Management/) cubre los patrones JIT, las sesiones grabadas, el aprovisionamiento efímero de credenciales y la relación con SPIFFE, Vault y los IdPs.

### Detectando el compromiso: ITDR

La prevención nunca es perfecta. **ITDR (Identity Threat Detection and Response)** es la categoría de producto que asume el compromiso como inevitable y se especializa en detectarlo: detección de *credential stuffing*, movimiento lateral apoyado en cuentas de servicio, ataques a Active Directory (Kerberoasting, DCSync, Golden Ticket), MFA bypass, tokens robados, impersonation anómala. Productos como Microsoft Defender for Identity, CrowdStrike Falcon Identity Protection, Silverfort y Semperis se integran con IdPs y directorios para correlacionar señales y emitir alertas accionables.

ITDR es la pieza que cierra el bucle: cuando detecta un compromiso, emite eventos que CAEP/Shared Signals propagan a los *relying parties* para revocar sesiones en tiempo real. El artículo sobre [Identity Threat Detection and Response](/posts/ITDR/) cubre las técnicas de detección, las fuentes de telemetría (logs de IdP, eventos de AD, tráfico de red) y la relación con SIEM, XDR y CAEP.

## De los roles a las políticas: el control de acceso

La autorización ha evolucionado a un ritmo propio, independiente de la autenticación. Tres modelos marcan el camino:

**RBAC (Role-Based Access Control)**, formalizado por el NIST en 1992 y estandarizado como INCITS 359-2012, agrupa permisos en roles y asigna roles a usuarios. Es simple, auditable y suficiente para la mayoría de las aplicaciones empresariales. Falla cuando el número de roles explota combinatoriamente o cuando las decisiones dependen del contexto.

**ABAC (Attribute-Based Access Control)**, estandarizado por NIST SP 800-162 en 2014, expresa la política como una función de atributos del sujeto, del recurso, de la acción y del contexto. Gana expresividad a costa de auditabilidad y rendimiento.

**ReBAC (Relationship-Based Access Control)**, popularizado tras el paper **Zanzibar** de Google (2019), modela la autorización como un grafo de relaciones tipadas entre sujetos y objetos. Es el modelo natural para dominios donde la autorización se define por jerarquías y *sharing* (documentos, proyectos, carpetas, tenants jerárquicos).

El artículo sobre [RBAC, ABAC y ReBAC](/posts/RBAC-ABAC-ReBAC/) analiza cuándo cada modelo es la respuesta correcta y cómo combinarlos.

## El estándar empresarial de autorización: XACML

Mucho antes de OPA, OpenFGA o Cedar, **XACML (eXtensible Access Control Markup Language, OASIS, versión 1.0 en 2003 y 3.0 en 2013)** formalizó la arquitectura de autorización empresarial con cuatro componentes que siguen siendo la referencia conceptual:

- **PEP (Policy Enforcement Point):** intercepta el acceso y aplica la decisión.
- **PDP (Policy Decision Point):** evalúa la política contra los atributos de la petición.
- **PAP (Policy Administration Point):** gestiona el ciclo de vida de las políticas.
- **PIP (Policy Information Point):** fuente externa de atributos para enriquecer la decisión.

XACML perdió tracción por su sintaxis XML pesada y por la aparición de DSLs más ergonómicos (Rego, Cedar, el DSL de OpenFGA), pero el modelo PEP/PDP/PAP/PIP sigue siendo el lenguaje común con el que todos los motores modernos se describen. El artículo sobre [XACML](/posts/XACML/) explica el flujo de decisión, las *combining algorithms* y por qué el modelo sobrevive a la sintaxis.

## Política como código: la era de los motores modernos

A partir de 2016 aparecen motores de política pensados para la era cloud-native:

**OPA (Open Policy Agent, 2016, CNCF Graduated en 2021)** introduce Rego como lenguaje de política inspirado en Datalog. Se despliega como daemon o sidecar, carga políticas en *bundles* y expone una API de decisión. Su integración nativa con Kubernetes mediante **Gatekeeper** lo ha convertido en el motor de política más usado en entornos cloud-native.

**CASBIN (2017)** es una librería multi-modelo configurable por ficheros PERM. Destaca por su flexibilidad y ligereza para casos *embedded* en aplicaciones.

**OpenFGA (Auth0/Okta, 2022, CNCF Sandbox)** es una implementación del modelo Zanzibar con tipado de relaciones y *userset rewrites*. Es la elección natural cuando la autorización es un grafo.

El artículo sobre [CASBIN, OPA y OpenFGA](/posts/CASBIN-OPA-OpenFGA/) compara los tres motores, los patrones de integración y los criterios de decisión según el dominio.

### Autorización como servicio gestionado: Entitlements as a Service

Los motores open source cubren el núcleo, pero operar autorización a escala (versionado de políticas, despliegue controlado, réplicas consistentes, tooling de *policy testing*, SDKs multi-lenguaje) es trabajo que muchos equipos prefieren no construir. A partir de 2019-2022 aparece la categoría *Entitlements as a Service*: productos que ofrecen autorización gestionada con SDK, decisiones con latencia sub-milisegundo y herramientas operacionales maduras.

**AuthZed/SpiceDB** (2021) es la implementación comercial de Zanzibar. **Cerbos** (2021) adopta un modelo de políticas declarativas en YAML con *conditional expressions*. **Permit.io** (2022) construye sobre OPA/OpenFGA añadiendo UI de gestión y *no-code policy editor*. **Oso** (2019, con Polar como DSL) se reposicionó hacia SaaS en 2023. El artículo sobre [Entitlements as a Service](/posts/Entitlements-as-a-Service/) compara las plataformas, los criterios de decisión frente a desplegar OPA o OpenFGA self-hosted, y los trade-offs de latencia, governance y vendor lock-in.

## Política en la plataforma: Kubernetes y los admission controllers

Kubernetes introduce un plano adicional de política: el **admission control**, ejecutado en el API server después de la autenticación y la autorización RBAC. Los *mutating* y *validating admission webhooks* permiten que sistemas externos decidan si un recurso entra al clúster y en qué forma.

**OPA Gatekeeper** lleva el motor de OPA al admission control mediante *ConstraintTemplates* y *Constraints*. **Kyverno** (CNCF Incubating) ofrece políticas declaradas en YAML nativo de Kubernetes, con capacidades de generación, mutación y verificación de imágenes (integración con Cosign para cadena de suministro). En la versión 1.30 llegó **ValidatingAdmissionPolicy** en el propio árbol de Kubernetes, basada en CEL.

Este plano es, en términos de XACML, un PEP global del clúster. El artículo sobre [Admission Control en Kubernetes](/posts/Admission-Control-Kubernetes/) compara Gatekeeper, Kyverno y CEL, y aborda políticas típicas (etiquetas obligatorias, registros permitidos, PSS, verificación de firmas).

## Patrones de seguridad en el perímetro y en el interior

En sistemas de microservicios la seguridad se distribuye en cuatro patrones complementarios:

- **API Gateway:** punto de entrada único que termina TLS, valida tokens, aplica rate limiting y enruta. Canónico para tráfico norte-sur.
- **BFF (Backend For Frontend):** pasarela especializada por tipo de cliente (móvil, web, partners) que resuelve la fan-out de necesidades distintas y aloja los *client secrets* de OAuth sin exponerlos a un SPA.
- **Service Mesh:** plano de control para tráfico este-oeste basado en sidecars (Istio, Linkerd, Consul Connect) o en eBPF (Cilium, ambient mesh). Proporciona mTLS automático, observabilidad, políticas L7 y resiliencia.
- **Sidecar proxy:** mecanismo que subyace al service mesh y a otros patrones de interceptación transparente.

El artículo sobre [API Gateway, BFF, Service Mesh y Sidecar](/posts/API-Gateway-BFF-Service-Mesh-Sidecar/) desarrolla la composición canónica (gateway en el borde + mesh en el interior) y la asignación de responsabilidades de seguridad entre capas.

## La confianza en el canal: mTLS

Entre servicios, los tokens *bearer* tienen una debilidad estructural: si alguien los roba, los puede usar desde cualquier sitio. **mTLS (mutual TLS)** cierra esa vulnerabilidad exigiendo que ambas partes de una conexión presenten certificados X.509 válidos. No es una capacidad nueva (TLS 1.0 ya la soportaba en 1999), pero su adopción masiva en microservicios llegó con los service meshes alrededor de 2017.

El problema de mTLS nunca fue el protocolo, sino la operativa: emitir, distribuir, rotar y revocar certificados a escala en entornos donde los pods son efímeros. Los service meshes automatizan esto; sin mesh, la complejidad explota. El artículo sobre [mTLS](/posts/mTLS/) cubre el handshake mutuo, los modos *strict* y *permissive*, la relación con SPIFFE y los modos de fallo habituales.

## La raíz de la confianza: PKI

Sin una CA, no hay certificados. Sin certificados, no hay mTLS. La **PKI (Public Key Infrastructure)** es la capa invisible que sostiene todo lo anterior: CA raíz, CAs intermedias, cadena de confianza, revocación (CRL, OCSP, *OCSP stapling*), restricciones de uso.

En Kubernetes la PKI interna firma certificados del API server, kubelets, etcd y tokens de ServiceAccount. **cert-manager** se ha convertido en el estándar *de facto* para la gestión del ciclo de vida de certificados, con *issuers* que hablan con Let's Encrypt, Vault o CAs privadas. El artículo sobre [PKI](/posts/PKI/) explica la arquitectura multi-nivel, la rotación de la CA raíz (el escenario que todos deben planear pero nadie quiere vivir) y el contexto de la migración post-cuántica.

### Donde viven las claves: HSM y KMS

La firma de un JWT, la emisión de un certificado, el cifrado de un secreto en reposo: todo eventualmente se reduce a operaciones criptográficas que usan una clave privada, y esa clave tiene que vivir en algún sitio. Un archivo en disco es la peor opción; un **HSM (Hardware Security Module)** que implementa PKCS#11 (Cryptoki) es la mejor. Los HSMs físicos (Thales, Utimaco, YubiHSM) garantizan que la clave nunca abandona el hardware, y los **KMS gestionados por cloud** (AWS KMS, GCP Cloud KMS, Azure Key Vault HSM, Google Cloud External Key Manager) ofrecen la misma garantía como servicio, con respaldo FIPS 140-2/3.

HSM y KMS son el fondo de todo lo anterior: sin ellos, las CAs no son de fiar, los JWTs son falsificables y los secretos cifrados son recuperables con un *dump* de disco. El artículo sobre [HSM y KMS](/posts/HSM-y-KMS/) desarrolla el modelo de *key hierarchy* (root, data encryption keys, key encryption keys), la rotación, el envelope encryption, y los criterios de decisión entre HSM dedicado, appliance cloud y KMS multi-tenant.

## Identidad de carga de trabajo: SPIFFE

En entornos dinámicos como Kubernetes, las IPs no son identidad estable y los secretos estáticos son una deuda que se acumula. **SPIFFE (Secure Production Identity Framework For Everyone, CNCF Graduated 2022)** estandariza el concepto de identidad criptográfica de workload: cada carga de trabajo obtiene un **SPIFFE ID** (un URI del tipo `spiffe://trust-domain/path`) respaldado por un **SVID** (SPIFFE Verifiable Identity Document) en formato X.509 o JWT.

**SPIRE** es la implementación de referencia. Su *Server* emite SVIDs, su *Agent* corre en cada nodo y verifica las *attestations* (qué es el workload, en qué nodo, con qué imagen) antes de entregar una identidad. SPIFFE elimina los secretos estáticos y habilita mTLS automático sin intervención manual.

El artículo sobre [SPIFFE y SPIRE](/posts/SPIFFE-SPIRE/) cubre las *attestations*, la Workload API, la federación entre dominios de confianza y cómo encaja con el service mesh.

## Secretos como servicio: Vault y la gestión dinámica

La identidad resuelve una parte del problema; la otra es qué puede pedir un workload con ella. **HashiCorp Vault** (2015) popularizó la idea de **secretos dinámicos**: en lugar de entregar una credencial fija, Vault genera credenciales bajo demanda con TTL corto (una password de base de datos que existe durante 10 minutos, una clave AWS temporal, un certificado recién firmado). El **transit engine** ofrece cifrado-como-servicio, y el **PKI engine** emite certificados dinámicamente para mTLS.

A su alrededor orbita un ecosistema: **External Secrets Operator** sincroniza secretos desde proveedores externos (Vault, AWS Secrets Manager, GCP Secret Manager, Azure Key Vault) hacia Kubernetes Secrets; **Sealed Secrets** permite comprometer secretos cifrados en Git; el **Secrets Store CSI Driver** los monta como volúmenes.

El artículo sobre [gestión de secretos](/posts/Secret-Management/) desarrolla estas piezas y la evolución hacia arquitecturas *zero-static-secrets*.

## Identidad de workload en la nube

Los hyperscalers resolvieron el mismo problema que SPIFFE a su manera. **AWS IRSA (IAM Roles for Service Accounts)** y el más reciente **EKS Pod Identity**, **GCP Workload Identity** y **Azure Workload Identity** comparten el mismo truco conceptual: el clúster Kubernetes expone un *issuer* OIDC, el token de ServiceAccount actúa como ID Token, y la IAM del cloud confía en ese *issuer* mediante federación OIDC.

El resultado es el mismo que SPIFFE en espíritu (identidad criptográfica de workload sin secretos estáticos) con la diferencia de que la implementación vive en el plano gestionado del cloud. El artículo sobre [Cloud Workload Identity](/posts/Cloud-Workload-Identity/) compara las tres aproximaciones y discute la federación cross-cloud.

## La cadena de identidad: Token Exchange

Cuando un servicio A llama a B que llama a C en nombre de un usuario, ¿qué identidad presenta cada salto? Reenviar el token original tiene problemas de *audience* y de alcance; usar *client credentials* pierde el contexto del usuario. **RFC 8693 (Token Exchange, enero 2020)** estandariza el intercambio de tokens con dos semánticas: **impersonation** (B actúa como A, sin rastro del delegante) y **delegation** (B actúa en nombre de A, con el claim `act` registrando la cadena).

El artículo sobre [Token Exchange](/posts/Token-Exchange/) cubre los casos de uso, los tipos de token intercambiables (Access Token, ID Token, JWT, SAML 2.0), la integración con IdPs y las implicaciones de auditoría.

### Impersonation vs delegación: la semántica que rompe auditorías

Token Exchange define dos semánticas distintas que suelen confundirse, con consecuencias serias en auditoría y cumplimiento. En **impersonation**, el servicio B recibe un token que lo presenta como si fuera el usuario original: los logs downstream ven al usuario, no a B, y no hay rastro del intermediario. En **delegación** (con el *claim* `act`, *actor*), B actúa en nombre del usuario pero preserva la cadena: los logs pueden reconstruir quién inició la acción y quién la ejecutó.

Elegir la semántica equivocada es un error común con consecuencias legales: GDPR, SOX, HIPAA exigen trazabilidad de actor, y un sistema que usa impersonation donde debería usar delegación pierde esa trazabilidad silenciosamente. El artículo sobre [impersonation vs delegación](/posts/Impersonation-y-Delegacion/) desarrolla la semántica del claim `act`, los patrones de chain-of-custody en cadenas de servicios, y las implicaciones de auditoría en entornos regulados.

## El futuro del estándar: OAuth 2.1 y más allá

OAuth 2.0 acumuló más de una década de erratas, *best current practices* y extensiones. **OAuth 2.1** (en proceso en el IETF) consolida todo: PKCE obligatorio, *Implicit* y *Resource Owner Password Credentials* eliminados, *redirect URI matching* estricto, rotación de *refresh tokens*. Alrededor giran estándares complementarios: **DPoP (Demonstrating Proof of Possession, RFC 9449, 2023)** para tokens *sender-constrained* sin mTLS; **PAR (RFC 9126, 2021)** para empujar parámetros de autorización por canal trasero; **RAR (RFC 9396, 2023)** para autorización granular estructurada; **FAPI 2.0** como perfil financiero.

Más allá aparece **GNAP (Grant Negotiation and Authorization Protocol, RFC 9635, 2024)** como candidato a sucesor, con un modelo cliente-iniciado, multi-ronda y multi-sujeto. No es "OAuth 3": es un protocolo distinto, con su propia curva de adopción. El artículo sobre [el futuro de OAuth](/posts/OAuth-Futuro/) detalla qué adoptar hoy, qué vigilar y cómo planificar la transición.

### Autorización gestionada por el usuario: UMA 2.0

OAuth asume que el *resource owner* es el mismo usuario que se autentica en el momento. El modelo rompe cuando un usuario quiere compartir un recurso con otro usuario: por ejemplo, compartir el historial médico con un especialista durante tres días, o delegar acceso a la contabilidad al gestor. **UMA 2.0 (User-Managed Access, Kantara Initiative, 2017)** extiende OAuth introduciendo un *Authorization Server* centrado en el usuario que gestiona políticas de compartición entre *resource owner* (quien posee el recurso) y *requesting party* (un tercero distinto).

UMA no ha ganado adopción masiva (OAuth planos con *scopes* genéricos cubre muchos casos "suficientemente bien") pero sigue siendo la respuesta correcta en dominios como sanidad, finanzas personales y SSI/wallets, donde el control de compartición es el requisito central. El artículo sobre [UMA 2.0](/posts/UMA2/) desarrolla el modelo de *Permission Tickets*, la diferencia con OAuth estándar y los casos donde el *resource owner* no es el solicitante.

## El horizonte: identidad descentralizada

Todo el modelo anterior asume un IdP central que emite y controla la identidad. Una línea paralela de investigación y estandarización cuestiona esa premisa: **Self-Sovereign Identity (SSI)** propone que el sujeto controle sus propias credenciales sin depender de un emisor intermediario. Los tres bloques centrales son **Decentralized Identifiers (DIDs, W3C Recommendation, 2022)** como identificadores persistentes no emitidos por una autoridad; **Verifiable Credentials (VC, W3C Recommendation v2.0, 2025)** como credenciales firmadas que el sujeto guarda y presenta selectivamente; y los **DID Methods** que anclan los identificadores a distintos sistemas (ledgers, web, clave pública).

El impulso regulatorio más claro es **eIDAS 2.0** (Reglamento UE 2024/1183) que obliga a los estados miembros a ofrecer una *European Digital Identity Wallet* antes de 2026 basada en estos estándares. Mientras OAuth/OIDC no desaparezcan en décadas, este horizonte define el siguiente escalón. El artículo sobre [Verifiable Credentials, DIDs y SSI](/posts/Verifiable-Credentials-DIDs-SSI/) cubre el modelo emisor-titular-verificador, los formatos JWT-VC y SD-JWT, los *presentation exchange* y el encaje con eIDAS 2.

## Mapa de la serie

| Concepto | Referencia / Año | Artículo |
|---|---|---|
| Autenticación y Autorización | Principio fundacional | [Los dos pilares](/posts/Autenticacion-y-Autorizacion/) |
| Kerberos | RFC 4120 / 1988, v5 2005 | [El abuelo de la autenticación empresarial](/posts/Kerberos/) |
| LDAP / Active Directory | RFC 4511 / 2006, AD 2000 | [El directorio empresarial](/posts/LDAP-Active-Directory/) |
| SAML 2.0 | OASIS / 2005 | [La federación empresarial](/posts/SAML/) |
| OAuth 2.0 | RFC 6749 / 2012 | [El protocolo de delegación](/posts/OAuth2/) |
| JWT | RFC 7519 / 2015 | [Anatomía y trampas](/posts/JWT/) |
| Macaroons | Google / 2014 | [La alternativa olvidada a JWT](/posts/Macaroons/) |
| OIDC | OpenID Foundation / 2014 | [La capa de identidad](/posts/OIDC/) |
| Sesiones y revocación | RFC 7662, OIDC Session Mgmt | [Ciclo de vida de tokens](/posts/Sesiones-y-Revocacion/) |
| PKCE | RFC 7636 / 2015 | [El eslabón de los clientes públicos](/posts/PKCE/) |
| Keycloak / Okta | 2013 / 2009 | [Anatomía del IdP moderno](/posts/Keycloak-Okta-IdP/) |
| CIAM | Auth0, Cognito, Stytch, WorkOS | [Identidad orientada a consumidores](/posts/CIAM/) |
| OpenID Federation | RFC 9678 / 2024 | [Federación de IdPs a escala](/posts/OpenID-Federation/) |
| SCIM | RFC 7643/7644 / 2015 | [Provisioning entre IdP y aplicaciones](/posts/SCIM/) |
| Identity Governance (IGA) | SailPoint, Saviynt, Omada | [Gobernanza sobre IAM](/posts/Identity-Governance-IGA/) |
| MFA | RFC 4226 / 2005, RFC 6238 / 2011 | [Factores y step-up authentication](/posts/MFA/) |
| FIDO2 / WebAuthn / Passkeys | FIDO 2018, W3C 2019, Passkeys 2022 | [El fin de la contraseña](/posts/FIDO2-WebAuthn-Passkeys/) |
| CAEP / Shared Signals | OpenID Foundation 2022-2024 | [Señales continuas de identidad](/posts/CAEP-Shared-Signals/) |
| Zero Trust | Forrester 2010, NIST SP 800-207 / 2020 | [La filosofía del perímetro roto](/posts/Zero-Trust/) |
| Privileged Access Management | CyberArk, Boundary, Teleport | [Acceso efímero y auditable](/posts/Privileged-Access-Management/) |
| ITDR | Defender for Identity, Silverfort, Semperis | [Detección de amenazas de identidad](/posts/ITDR/) |
| RBAC / ABAC / ReBAC | NIST 1992 / NIST 2014 / Zanzibar 2019 | [La evolución del control de acceso](/posts/RBAC-ABAC-ReBAC/) |
| XACML | OASIS / 2003, v3 2013 | [El modelo PEP/PDP/PAP/PIP](/posts/XACML/) |
| CASBIN / OPA / OpenFGA | 2017 / 2016 / 2022 | [Motores de política modernos](/posts/CASBIN-OPA-OpenFGA/) |
| Entitlements as a Service | AuthZed, Cerbos, Permit, Oso | [Autorización gestionada](/posts/Entitlements-as-a-Service/) |
| Admission Control K8s | Gatekeeper, Kyverno | [Política en la plataforma](/posts/Admission-Control-Kubernetes/) |
| API Gateway / BFF / Service Mesh | Patrones cloud-native | [Seguridad en el perímetro y el interior](/posts/API-Gateway-BFF-Service-Mesh-Sidecar/) |
| mTLS | TLS 1.0 / 1999, adopción 2017+ | [Confianza mutua en el canal](/posts/mTLS/) |
| PKI | RFC 5280 / 2008 | [La raíz de la confianza](/posts/PKI/) |
| HSM y KMS | PKCS#11, FIPS 140-2/3 | [Donde viven las claves](/posts/HSM-y-KMS/) |
| SPIFFE / SPIRE | CNCF / 2017 | [Identidad de workload](/posts/SPIFFE-SPIRE/) |
| Secret Management | Vault 2015, ESO, Sealed Secrets | [Gestión dinámica de secretos](/posts/Secret-Management/) |
| Cloud Workload Identity | IRSA, GCP WI, Azure WI | [Los SPIFFE de los hyperscalers](/posts/Cloud-Workload-Identity/) |
| Token Exchange | RFC 8693 / 2020 | [Delegación de identidad entre servicios](/posts/Token-Exchange/) |
| Impersonation vs delegación | Claim `act`, chain-of-custody | [La semántica de auditoría](/posts/Impersonation-y-Delegacion/) |
| OAuth 2.1, DPoP, GNAP | RFC 9449 / 2023, RFC 9635 / 2024 | [El futuro del estándar](/posts/OAuth-Futuro/) |
| UMA 2.0 | Kantara Initiative / 2017 | [Autorización gestionada por el usuario](/posts/UMA2/) |
| Verifiable Credentials / DIDs / SSI | W3C 2022/2025, eIDAS 2.0 / 2024 | [La identidad descentralizada](/posts/Verifiable-Credentials-DIDs-SSI/) |

## Conclusión

Ninguno de estos conceptos existe en el vacío. OAuth necesita JWT para escalar, JWT necesita PKI para firmar, PKI necesita una autoridad que emita y rote certificados, y todo eso se apoya en una identidad de workload que SPIFFE o los cloud providers proporcionan. La autorización, a su vez, asume una autenticación ya resuelta y una identidad verificable que transportar entre capas. Zero Trust es el marco que obliga a que cada capa verifique sin confiar en las anteriores.

La seguridad moderna no es una suma de productos, es un tejido de protocolos que colaboran. Cada artículo de esta serie profundiza en una de esas hebras. Se pueden leer en cualquier orden, pero volver a este mapa de vez en cuando ayuda a no perder la perspectiva del conjunto.
