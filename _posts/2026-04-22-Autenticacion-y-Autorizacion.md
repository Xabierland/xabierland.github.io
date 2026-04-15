---
title: Autenticación y autorización: los dos pilares que los arquitectos confunden
author: Xabierland
description: >-
    Autenticación y autorización son dos procesos distintos que responden a preguntas diferentes: quién eres y qué puedes hacer. Confundirlos en el diseño produce código de seguridad disperso, auditorías imposibles y fugas entre tenants. Entender la separación es el primer paso para construir cualquier arquitectura moderna de seguridad.
date: 2026-04-22 10:00
categories: [DevSecOps, Seguridad de Aplicaciones]
tags: [Autenticación, Autorización, Seguridad, Arquitectura, Identidad]
---

Pocas confusiones han causado tantos incidentes de seguridad como tratar autenticación y autorización como un único problema resuelto por el mismo componente. La industria lleva décadas repitiendo el mantra "AuthN y AuthZ son cosas distintas" y, sin embargo, cada revisión arquitectónica descubre lógica de negocio comprobando `user.role == "admin"` al lado de una consulta SQL. Este artículo forma parte de la serie [La evolución de la seguridad en aplicaciones modernas](/posts/La-evolucion-de-la-seguridad-en-aplicaciones-modernas/).

## Por qué la distinción existe

La separación entre autenticación y autorización no es una obsesión académica, es una consecuencia directa de cómo evolucionaron los sistemas multiusuario. En los primeros mainframes, la cuenta de usuario y los permisos eran el mismo registro. Cuando el sistema operativo comprobaba tu login también decidía qué podías ejecutar. Esto funcionaba porque el universo de usuarios era cerrado, pequeño y administrado por una única autoridad.

El modelo AAA (Authentication, Authorization, Accounting), formalizado por el IETF en RFCs como RFC 2903 y RFC 3539 para entornos de red, fue uno de los primeros intentos formales de separar estas tres responsabilidades. RADIUS primero y después Diameter demostraron que un servidor podía autenticar sin decidir nada sobre permisos, delegando esa decisión a otra capa. Esa separación fue lo que permitió que un mismo proveedor de identidad sirviera a múltiples aplicaciones con políticas de acceso radicalmente distintas.

Cuando la web empujó a las aplicaciones a un mundo abierto, con millones de identidades potenciales y delegación entre dominios, la falta de separación se volvió catastrófica. Las aplicaciones que mezclaban ambas responsabilidades no podían federar identidad con terceros, no podían evolucionar sus modelos de permisos sin tocar el flujo de login, y no podían auditar una decisión sin reconstruir todo el camino del código.

## Dos preguntas, dos respuestas

La autenticación responde a una única pregunta: ¿puedo demostrar que quien se presenta es la entidad que dice ser? Es un proceso de verificación de una afirmación de identidad respaldada por una prueba: contraseña, factor biométrico, token criptográfico, certificado, posesión de una clave privada. Su resultado es booleano en el fondo, aunque el artefacto de salida (un JWT firmado, una cookie de sesión, un ticket Kerberos) transporte también atributos del sujeto.

La autorización responde a una pregunta completamente distinta: dado un sujeto ya identificado, ¿puede realizar esta acción concreta sobre este recurso concreto en este contexto concreto? No es una propiedad intrínseca del sujeto, es una función de sujeto, acción, recurso y contexto. El mismo usuario puede estar autorizado a leer un documento a las diez de la mañana desde la oficina y no autorizado a las dos de la madrugada desde una IP extranjera.

### Identidad y verificación de identidad

Dentro de la autenticación conviene distinguir entre identidad e *identity proofing*. La identidad es el conjunto de atributos que describen a un sujeto en el sistema. El *identity proofing* es el proceso previo por el cual esa identidad se vincula de forma fiable a una persona o entidad del mundo real, normalmente durante el alta. NIST SP 800-63 (National Institute of Standards and Technology) formaliza esto en tres niveles de aseguramiento: IAL para el *proofing*, AAL para el acto de autenticación y FAL para la federación. Esa descomposición explica por qué "iniciar sesión con Google" puede ser una autenticación fuerte y una identidad débil a la vez: el factor es robusto, pero el vínculo con una persona real depende de lo que Google haya verificado.

### El artefacto de autenticación como contrato

Cuando la autenticación termina con éxito, el sistema emite un artefacto verificable: un token, un ticket, una cookie firmada. Ese artefacto es el contrato que viaja hasta las capas siguientes. Un JWT (JSON Web Token) definido en RFC 7519 (JSON Web Token) transporta *claims* sobre el sujeto, pero no es una autorización. Confundir los *claims* con la decisión de autorización es el error de diseño que más código de seguridad ha dispersado por las aplicaciones.

## El error clásico: acoplarlas en el código de negocio

El antipatrón más común aparece cuando la lógica de negocio toma decisiones de autorización directamente. Es habitual encontrar condiciones del tipo "si el rol es administrador, muestra el botón" repartidas por *controllers*, *views*, *middlewares* y consultas a base de datos. El problema no es que la comprobación exista, es que cada una de esas comprobaciones es una política implícita enterrada en el código fuente.

Las consecuencias se acumulan con el tiempo. La primera es la imposibilidad de auditoría. Nadie puede responder con certeza a la pregunta "¿quién puede borrar facturas?" porque la respuesta está dispersa en decenas de archivos. La segunda es la fuga entre tenants. Cuando la aplicación multi-tenant comprueba el rol pero no el tenant al que pertenece el recurso, basta un identificador manipulado en la URL para acceder a datos ajenos. La tercera es la rigidez evolutiva. Cambiar de un modelo basado en roles a uno basado en atributos o relaciones obliga a reescribir cada punto de decisión.

> Si la respuesta a "¿quién puede hacer X?" no se puede dar sin buscar en el código, la autorización no está diseñada, está sucediendo por accidente.
{: .prompt-warning }

## El principio de separación

La regla operativa es sencilla de enunciar y difícil de imponer: autenticar en el borde, autorizar en cada capa. El borde (API Gateway, *ingress*, BFF) es donde se valida que el portador del token es quien dice ser y se normaliza la identidad antes de entrar al sistema. A partir de ahí, cada capa que tenga una decisión de acceso la toma consultando un motor de política, no calculándola.

Este planteamiento produce arquitecturas donde la lógica de autenticación vive en componentes especializados (un IdP como Keycloak u Okta), los tokens viajan como vehículo de identidad, y la autorización se evalúa contra políticas externalizadas (OPA, Casbin, OpenFGA, XACML). El código de negocio queda limpio de condicionales de seguridad y las decisiones son inspeccionables, versionables y auditables.

### Puntos de decisión y puntos de aplicación

La terminología que usan XACML y la arquitectura Zero Trust ayuda a pensar esto con claridad. El PDP (Policy Decision Point) es el componente que evalúa una política y devuelve una decisión. El PEP (Policy Enforcement Point) es el componente que intercepta la solicitud y aplica la decisión. La autenticación produce el sujeto que entra al PDP como entrada. La autorización es lo que el PDP calcula. Confundir PEP y PDP es lo que convierte el *middleware* de autenticación en un nido de reglas de autorización.

## Cuándo se puede tolerar la mezcla y cuándo no

No todas las aplicaciones necesitan esta separación con la misma severidad. Un script interno con tres usuarios administradores puede tolerar una comprobación inline sin mayor problema. El cálculo cambia cuando aparece cualquiera de estas señales: más de un *tenant*, requisitos regulatorios (SOC 2, ISO 27001, GDPR), federación con terceros, necesidad de emitir auditorías reproducibles o cualquier operación sobre datos de terceros bajo responsabilidad legal.

El síntoma de que la mezcla ya está costando dinero suele ser el mismo: cada cambio de permisos dispara una ola de regresiones en funcionalidad no relacionada, porque el código de autorización está trenzado con el flujo de negocio.

> Cuando el equipo de producto pide un permiso nuevo y el equipo de seguridad pide una semana de análisis, el problema no es la política, es la arquitectura.
{: .prompt-tip }

### Modos de fallo habituales

El primer modo de fallo es la *authorization by obscurity*: la aplicación no muestra el botón, pero el *endpoint* responde si lo invocas directamente. El segundo es el *IDOR* (Insecure Direct Object Reference): se comprueba que el usuario está autenticado pero no que el objeto referenciado le pertenece. El tercero es el *role explosion*: cuando la granularidad de permisos sobrepasa lo que un modelo basado en roles puede expresar sin crear cientos de roles sintéticos. Cada uno de estos fallos desaparece o se vuelve muy improbable cuando la autorización se externaliza a un motor de políticas con reglas explícitas.

## Cómo encaja con el resto de la arquitectura

La separación entre autenticación y autorización es la base sobre la que se apoyan todas las piezas modernas de la serie. Los flujos OAuth 2.0 (Open Authorization) y OIDC (OpenID Connect) separan explícitamente la autenticación del usuario, la emisión del *access token* y la decisión de permitir una acción en el recurso. Los motores como OPA o Casbin solo tienen sentido si el sujeto llega ya autenticado. Zero Trust convierte esta separación en principio de diseño al exigir verificación continua de ambos ejes. Y cualquier estrategia de federación (SAML, Kerberos, mTLS, SPIFFE) se apoya en que la identidad que llega sea confiable antes de tocar una política.

## Artículos relacionados en esta serie

- [OAuth 2.0](/posts/OAuth2/)
- [Keycloak, Okta y los IdP modernos](/posts/Keycloak-Okta-IdP/)
- [RBAC, ABAC y ReBAC](/posts/RBAC-ABAC-ReBAC/)
- [Zero Trust](/posts/Zero-Trust/)

## Referencias

- RFC 2903 (agosto 2000). *Generic AAA Architecture*.
- RFC 3539 (junio 2003). *Authentication, Authorization and Accounting (AAA) Transport Profile*.
- RFC 7519 (mayo 2015). *JSON Web Token (JWT)*.
- NIST SP 800-63 (2017). *Digital Identity Guidelines*.
- OWASP (2024). *Application Security Verification Standard (ASVS)*, capítulos V2 y V4.
