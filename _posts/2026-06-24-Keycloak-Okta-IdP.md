---
title: "Keycloak y Okta: anatomía de un proveedor de identidad moderno"
author: Xabierland
description: >-
    Un proveedor de identidad moderno no es un login con esteroides, es un componente que concentra autenticación, federación, ciclo de vida de usuarios, MFA y políticas. Entender qué comparten Keycloak y Okta, en qué se diferencian de Auth0, Entra ID o Cognito, y qué problemas no resuelve un IdP es condición previa para elegir uno.
date: 2026-06-24 10:00
categories: [DevSecOps, Seguridad de Aplicaciones]
tags: [Keycloak, Okta, IdP, OAuth2, OIDC, SAML, Identidad]
---

Pocos componentes concentran tanto riesgo operativo y arquitectónico como el IdP (Identity Provider). Una decisión que parece técnica, entre Keycloak o Okta, determina durante años el modelo de extensibilidad, los costes por usuario activo, la disponibilidad del login de toda la organización y los mecanismos de federación disponibles. Este artículo forma parte de la serie [La evolución de la seguridad en aplicaciones modernas](/posts/La-evolucion-de-la-seguridad-en-aplicaciones-modernas/).

## Por qué centralizar la identidad

El problema que resuelve un IdP es el de tener una única autoridad sobre quién es un usuario y cómo se autentica, mientras las aplicaciones y servicios que dependen de esa identidad se cuentan por decenas o cientos. En una organización sin IdP, cada aplicación gestiona su propia base de usuarios, sus propias políticas de contraseñas, su propio MFA (Multi-Factor Authentication) y su propia integración con directorios corporativos. El resultado es conocido: contraseñas inconsistentes, bajas de empleados que no se propagan, auditorías imposibles y un coste de mantenimiento que crece linealmente con el número de aplicaciones.

La consolidación en un IdP centraliza cinco responsabilidades. Primero, la autenticación de usuarios contra un único almacén o federación de ellos. Segundo, la federación con identidades externas (otro IdP corporativo, proveedor social, SAML de un partner). Tercero, el ciclo de vida de usuarios: alta, baja, sincronización con RRHH, *just-in-time provisioning*. Cuarto, los factores adicionales: MFA, biometría, llaves FIDO2. Quinto, las políticas de acceso condicionado: qué origen, qué dispositivo, qué hora, qué nivel de aseguramiento exige cada aplicación.

## Conceptos compartidos entre IdPs

Aunque la terminología varía, casi todos los IdPs comparten un mismo conjunto de abstracciones. Entenderlas desacopladas de la marca ayuda a comparar productos y a migrar entre ellos sin rehacer la arquitectura.

### Realm o tenant

La unidad de aislamiento superior. Un *realm* (Keycloak) o *tenant* (Okta, Auth0) es una frontera administrativa donde viven usuarios, aplicaciones, roles y políticas independientes. La decisión de usar un único *realm* para toda la organización o segmentarlo por unidad de negocio, marca o entorno condiciona la topología de federación y la granularidad de administración.

### Cliente o aplicación

Cada aplicación que delega la autenticación en el IdP se registra como un *client*. El *client* define su tipo (público, confidencial), los *redirect_uri* permitidos, los *grant types* habilitados, las *scopes* que puede solicitar y los *claim mappers* que personalizan qué atributos viajarán en el *token*.

### Scopes y claim mappers

Los *scopes* son el contrato de qué atributos o permisos puede pedir el cliente. Los *claim mappers* son las reglas que materializan esos atributos en el token: lectura de un campo del directorio, cálculo de una pertenencia a grupo, enriquecimiento con datos externos. La calidad de los *claims* emitidos determina si la aplicación puede tomar decisiones de autorización sin volver a consultar el IdP.

### Flujos de autenticación

Un flujo (*authentication flow*) es la secuencia configurable de pasos por los que pasa un usuario hasta completar un login: formulario de usuario y contraseña, reto MFA, validación de dispositivo, aceptación de términos, *captcha*. Keycloak los expone como árbol editable de ejecuciones; Okta los llama *sign-on policies* y *authenticators*. La potencia de esta abstracción es que permite modificar la experiencia de login sin tocar las aplicaciones.

### Identity brokering

La federación entrante. El IdP actúa como intermediario: delega la autenticación a otro proveedor (Google, Microsoft, un SAML corporativo, un OIDC externo) y, tras recibir la identidad, la normaliza, la mapea a un usuario interno (creándolo al vuelo si procede) y emite sus propios *tokens*. Desde la perspectiva de las aplicaciones, siempre hablan con el mismo IdP, aunque la raíz de la autenticación esté en otro sitio.

## Keycloak

Keycloak nació en Red Hat en 2014 como evolución de PicketLink, y desde 2023 es un proyecto CNCF *incubating*. Es de código abierto (licencia Apache 2.0), se despliega sobre Java y actualmente sobre Quarkus (desde Keycloak 17, 2022), y ofrece un *Kubernetes Operator* oficial que gestiona despliegue, *realms* declarativos y actualizaciones.

Su punto fuerte es la extensibilidad. Los SPI (Service Provider Interface) permiten reemplazar o extender casi cualquier comportamiento: *authenticators* personalizados, *user storage providers* para conectar directorios propios, *event listeners* para integrar con SIEM, *protocol mappers* específicos. Esa flexibilidad es valiosa en organizaciones con casos atípicos (cumplimiento regulatorio específico, integraciones con sistemas *legacy*, políticas de emisión de *claims* no triviales).

Su *multi-tenancy* se articula alrededor de los *realms*. Cada *realm* es independiente, pero comparten infraestructura. Para SaaS con miles de *tenants* el modelo puede tensionar la base de datos y obliga a arquitecturas dedicadas; para organizaciones con decenas de *realms* funciona sin sobresaltos.

La contrapartida es operativa. Autoalojar Keycloak implica asumir la responsabilidad de disponibilidad, actualizaciones, *tuning* de base de datos, caché distribuido (Infinispan), y plan de recuperación ante desastres. No es trivial: Keycloak ha sido históricamente sensible al tamaño de *sessions* y *caches*, y sus upgrades entre versiones mayores requieren lectura atenta de las notas de migración.

## Okta

Okta se fundó en 2009 como SaaS de identidad y ha crecido hasta convertirse en referencia del mercado empresarial. Su propuesta es la opuesta a Keycloak: no hay infraestructura que operar. Se paga por MAU (Monthly Active Users) y el servicio absorbe la complejidad de disponibilidad, escalado y certificaciones.

Su pieza central es el Universal Directory, un almacén de usuarios flexible que admite atributos personalizados y múltiples fuentes. Las *sign-on policies* combinan reglas de red, dispositivo y factor exigido. Las *hooks* (inline y event) permiten extender el comportamiento llamando a endpoints externos durante el flujo, cumpliendo un papel parecido al de los SPI de Keycloak pero desde fuera.

En 2021 Okta adquirió Auth0. Aunque ambas pertenecen al mismo grupo, siguen siendo productos diferenciados. Auth0 mantiene una propuesta orientada al desarrollador, con *Actions* en JavaScript, buena experiencia de *onboarding* y un modelo de *tenants* pensado para equipos de producto. Okta Workforce Identity Cloud y Okta Customer Identity Cloud (esta última basada en Auth0) cubren respectivamente el caso interno de la plantilla y el caso de clientes finales.

## Otros protagonistas

Microsoft Entra ID (antes Azure AD, rebautizado en 2023) domina el caso corporativo cuando la organización ya vive en Microsoft 365. AWS Cognito es frecuente en cargas de trabajo AWS donde se valora la integración nativa, aunque su flexibilidad como IdP de propósito general es limitada. Google Cloud Identity y Google Identity Platform cubren el caso consumidor con fuerte integración con el ecosistema Google. Ping Identity y ForgeRock (ahora parte de Thoma Bravo, fusionados con Ping en 2023) compiten en el segmento empresarial con IdPs robustos y largos historiales en banca y telecomunicaciones.

## El IdP como confluencia de tres roles

Un IdP moderno desempeña simultáneamente tres papeles que conviene distinguir. Es *Authorization Server* en el sentido de OAuth 2.0: emite *access tokens* y *refresh tokens*. Es *OpenID Provider* en el sentido de OIDC: emite *ID tokens* con identidad autenticada y expone el *UserInfo endpoint*. Es *SAML IdP* para aplicaciones que consumen SAML 2.0 (OASIS, 2005), todavía imprescindible en muchos SaaS empresariales. El mismo producto atiende los tres roles con protocolos distintos y, a menudo, el mismo usuario se autentica una sola vez pero obtiene artefactos en varios formatos según la aplicación.

> Elegir un IdP es elegir, al mismo tiempo, un Authorization Server OAuth, un OIDC Provider y un SAML IdP. Si alguno de los tres no encaja con tus aplicaciones, la decisión está mal tomada.
{: .prompt-info }

## Lo que un IdP no hace

La confusión más costosa es pensar que adoptar un IdP resuelve la autorización. No la resuelve. Un IdP puede transportar *roles* o *groups* en los *tokens*, pero la decisión de si un usuario puede realizar una acción concreta sobre un recurso concreto pertenece a un motor de autorización (OPA, Casbin, OpenFGA) o a la lógica de la aplicación. Confundir *claims* con políticas es el error que mantiene durante años reglas de autorización dispersas por decenas de aplicaciones.

Tampoco es el componente que gestiona identidades de carga de trabajo. Los servicios que se autentican entre sí necesitan certificados, *secrets* o *SPIFFE SVIDs*, y eso lo cubren otros componentes (*secret managers*, SPIRE, el proveedor de nube). Pedirle a un IdP que gestione identidad de servicios fuerza abstracciones que no están pensadas para ciclos de vida cortos, rotación continua y volúmenes altos.

## Criterios de decisión

La elección entre self-hosted y SaaS es rara vez técnica; es sobre todo organizativa. Self-hosted (Keycloak y derivados) gana cuando hay requisitos estrictos de soberanía de datos, cuando la organización tiene ingeniería operativa para asumirlo y cuando el coste por MAU a escala no es asumible. SaaS (Okta, Auth0, Entra ID, Cognito) gana cuando el tiempo de puesta en marcha importa, cuando las certificaciones (SOC 2, ISO 27001, FedRAMP) deben venir preintegradas y cuando el equipo de seguridad prefiere firmar un SLA.

Los criterios operativos a contrastar son seis: cumplimiento y residencia de datos; extensibilidad necesaria (SPI, hooks, *actions*); coste total frente a MAU previstos a tres años; *vendor lock-in* y coste de salida; SLAs ofrecidos frente a SLO internos del login; soporte a los protocolos de las aplicaciones existentes (OIDC, SAML, LDAP, WS-Federation para *legacy*).

> El coste real de un IdP SaaS no es su precio por MAU, es lo que cuesta migrar si la negociación del próximo contrato se tuerce.
{: .prompt-warning }

## Patrones de integración

El *identity brokering* merece atención. Conectar el IdP central a un LDAP o Active Directory corporativo permite no migrar las credenciales y mantener el SSO corporativo intacto. Conectar proveedores sociales (Google, Apple, GitHub) habilita *consumer login* sin desarrollar el flujo. Conectar IdPs externos vía SAML u OIDC permite federar con partners sin abrirles la base de usuarios propia. Cada federación añade un eslabón que hay que monitorizar: un fallo en el IdP federado se percibe como fallo del IdP propio.

## Alta disponibilidad y radio de impacto

Un IdP caído es un login caído, y un login caído es toda la organización parada. Cualquier despliegue serio exige redundancia activa-activa, base de datos replicada, caché distribuido consistente y pruebas regulares de *failover*. Para Okta y otros SaaS esto se delega en el proveedor; para Keycloak hay que diseñarlo. Históricamente, los mayores incidentes han venido no de caídas de infraestructura, sino de *tokens* que dejaron de firmar correctamente tras una rotación de claves mal coordinada con los *caches* de las aplicaciones. Rotar claves de firma sin romper nada es un arte que merece runbook dedicado.

## Artículos relacionados en esta serie

- [OAuth 2.0](/posts/OAuth2/)
- [OpenID Connect](/posts/OIDC/)
- [SAML](/posts/SAML/)
- [Token Exchange](/posts/Token-Exchange/)
- [Autenticación y autorización](/posts/Autenticacion-y-Autorizacion/)

## Referencias

- RFC 6749 (octubre 2012). *The OAuth 2.0 Authorization Framework*.
- OpenID Foundation (2014). *OpenID Connect Core 1.0*.
- OASIS (2005). *Security Assertion Markup Language (SAML) V2.0*.
- Red Hat / CNCF (2023). *Keycloak Documentation y Keycloak Operator*.
- Okta. *Okta Developer Documentation*.
- NIST SP 800-63-3 (2017). *Digital Identity Guidelines*.
