---
title: "Token Exchange: cómo los servicios delegan y transforman identidad en arquitecturas complejas"
author: Xabierland
description: >-
    En cadenas de servicios donde A llama a B y B llama a C, propagar la identidad del usuario sin romper el principio de mínimo privilegio exige algo más que reenviar tokens. RFC 8693 define Token Exchange como el mecanismo estándar para que un servicio solicite un nuevo token derivado, distinguiendo entre impersonación y delegación, y permitiendo traducir entre OAuth y SAML.
date: 2026-07-22 10:00
categories: [DevSecOps, Seguridad de Aplicaciones]
tags: [Token Exchange, OAuth2, RFC 8693, Delegation, JWT, Microservicios]
---

El problema aparece en cuanto una arquitectura de microservicios pasa de dos saltos. El usuario llama a un BFF, el BFF llama a un servicio de pedidos, y el servicio de pedidos necesita llamar al servicio de inventario con contexto de usuario. ¿Qué token usa el servicio de pedidos para llamar al inventario? Este artículo forma parte de la serie [La evolución de la seguridad en aplicaciones modernas](/posts/La-evolucion-de-la-seguridad-en-aplicaciones-modernas/).

## Por qué hizo falta un estándar

Durante años las arquitecturas distribuidas resolvieron la propagación de identidad de tres formas, todas malas. La primera, reenviar el *access token* original tal cual: el token emitido para el BFF viaja por dentro de la malla hasta el servicio de inventario. El problema es doble. Por un lado, el *audience* del token (el *claim* `aud` en un JWT según RFC 7519) se emitió para el BFF o para un conjunto limitado de servicios, y aceptarlo en cualquier otro destino relaja la validación hasta romperla. Por otro, el *scope* del token suele ser amplio (las necesidades agregadas de la llamada completa), y reenviarlo significa dar al servicio interno más permisos de los que necesita. Es el antipatrón del *scope bloat*.

La segunda solución frecuente consistía en que cada servicio se autenticara con *client credentials* (una identidad de servicio OAuth) y enviara el identificador del usuario como parámetro aplicativo (`user_id` en un header o *claim* propio). Esta aproximación rompe el modelo de emisión: el servicio destinatario ya no verifica que un *Authorization Server* haya autenticado realmente al usuario; se fía de que el llamante diga la verdad. Se pierde la trazabilidad criptográfica y la auditoría se vuelve circunstancial.

La tercera era inventar un protocolo propio, firmar un JWT interno con una clave compartida y llamarlo *internal token*. Funcionaba, hasta que había que federar con un tercero, hacer *rotate* de la clave o explicarle a un auditor por qué el sistema interno no cumplía OAuth.

El IETF formalizó una respuesta estándar en RFC 8693 (enero de 2020): *OAuth 2.0 Token Exchange*.

## El mecanismo de RFC 8693

Token Exchange define una extensión del endpoint de *token* del *Authorization Server* con un nuevo *grant type*: `urn:ietf:params:oauth:grant-type:token-exchange`. El cliente presenta un token existente (el *subject_token*) y pide al Authorization Server que le emita uno nuevo con propiedades distintas: otra audiencia, otro *scope*, otro tipo, o añadiendo información sobre quién está actuando.

Los parámetros principales son pocos y conviene tenerlos presentes. `subject_token` y `subject_token_type` identifican el token de entrada que representa al sujeto original. `actor_token` y `actor_token_type`, opcionales, identifican a quien está actuando en nombre del sujeto (por ejemplo, el servicio intermedio). `audience` y `resource` indican el destinatario previsto del nuevo token. `scope` reduce o especifica los permisos. `requested_token_type` indica si se quiere un *access token*, un *id_token*, un JWT genérico o una aserción SAML.

La respuesta es un token nuevo, emitido por el *Authorization Server*, firmado por él, con las propiedades solicitadas si la política lo permite. Esa mediación del AS es lo que diferencia *Token Exchange* de un reenvío simple: la decisión de que un servicio pueda obtener un token derivado es explícita y auditable.

## Impersonación frente a delegación

RFC 8693 distingue con cuidado dos semánticas. La diferencia no es técnica sino de modelo de responsabilidad, y confundirlas convierte cualquier auditoría en una pesadilla.

### Impersonación

En el modo de impersonación, el servicio intermedio (B) obtiene un token que representa al sujeto original (A) sin rastro de que hubo intermediario. El servicio destino (C) ve el token como si A hubiera llamado directamente. El *claim* `act` no aparece. Este modo es simple y útil cuando C no necesita saber quién intermedió, y B actúa con todos los derechos del usuario sin restricciones.

El riesgo es obvio: si el token robado a B se usa contra C, no hay manera de distinguir una llamada legítima de una intrusión. La impersonación es aceptable solo cuando el nivel de confianza entre B y el AS es alto, y cuando la auditoría no requiere reconstruir la cadena.

### Delegación

En el modo de delegación, el token nuevo preserva al sujeto original y añade un *claim* `act` (actor) con la identidad del intermediario. Si hay varios saltos, el `act` se anida formando una cadena: el `act` del token tiene a su vez un `act` que representa al delegante anterior. El servicio destino puede aplicar políticas que dependan tanto del sujeto como del actor: "acepto esta operación solo si el usuario X la solicita y el servicio Y es quien intermedia".

> La cadena de `act` es lo que convierte Token Exchange en un mecanismo auditable. Si tu auditoría necesita responder "quién hizo qué pasando por dónde", la delegación es la única opción válida.
{: .prompt-info }

## Transformaciones entre tipos de token

Uno de los poderes menos aprovechados de Token Exchange es la traducción entre tipos. Los tipos estandarizados son `urn:ietf:params:oauth:token-type:access_token`, `id_token`, `refresh_token`, `jwt`, `saml1` y `saml2`. El AS puede aceptar una aserción SAML de entrada y emitir un JWT de salida, o viceversa. Esto resuelve un problema real: aplicaciones OAuth/OIDC modernas que necesitan llamar a servicios *legacy* que solo aceptan SAML, o al revés, sin duplicar infraestructura de autenticación.

El mismo mecanismo permite pasar de un *id_token* (identidad) a un *access_token* (autorización) con el *scope* concreto que la operación requiere, evitando que los clientes acumulen *access tokens* amplios por si acaso.

## Reducción de scope y restricción de audiencia

Dos usos que compensan por sí solos desplegar Token Exchange son la reducción de *scope* y la restricción de *audience*. El primero permite que un servicio intermedio tome un token con *scopes* amplios y derive un token con solo los *scopes* que va a usar en la siguiente llamada. El segundo asegura que el token derivado tenga como `aud` exclusivamente el servicio destino, de modo que si se filtra no sirve contra nadie más.

Combinar ambos da el patrón de mínimo privilegio extremo a extremo: cada salto lleva un token con el *scope* mínimo y la *audience* mínima necesarios para esa llamada concreta. El precio es una ronda adicional al AS por salto, que en arquitecturas de alto volumen puede requerir caché con mucho cuidado (evitando convertir la caché en un reemisor sin mediación).

## Casos de uso arquitectónicos

El BFF que convierte la sesión de cookie en un *access token* con contexto de usuario es el ejemplo canónico, especialmente cuando el BFF recibe de un *Authorization Server* un token amplio y necesita derivar tokens específicos por microservicio invocado.

La propagación de identidad a través de fronteras de confianza es otro. Cuando una llamada cruza de un *namespace* Kubernetes a otro, o de un dominio administrativo a otro, Token Exchange permite que la identidad se materialice en un token nuevo firmado por la autoridad del dominio receptor, en lugar de obligar a ambos dominios a confiar en el mismo emisor.

La integración entre mundos OAuth y SAML aparece sobre todo en migraciones. Una plataforma moderna en OAuth que debe seguir consumiendo APIs empresariales protegidas por SAML puede usar Token Exchange como puente en un único componente, en lugar de mantener dos flujos paralelos de autenticación.

## Implementaciones en producción

Keycloak implementa Token Exchange desde hace años, aunque durante tiempo fue una característica marcada como "technology preview" que requería habilitarse explícitamente. A partir de versiones recientes la implementación se considera estable y cubre impersonación y delegación con la sintaxis de RFC 8693 y ciertas extensiones propias.

Okta dispone del *Token Exchange grant type* en sus productos *Identity Engine*, sujeto a suscripción. Auth0 soporta *Custom Token Exchange* desde 2022, permitiendo aceptar *tokens* externos y derivar los propios bajo reglas configuradas. Microsoft Entra ID y AWS Cognito no implementan RFC 8693 literalmente, pero ofrecen mecanismos equivalentes bajo nombres distintos (On-Behalf-Of en Entra ID, que predata a RFC 8693 y tiene semántica parecida pero no idéntica).

## Errores comunes

El primero es tratar Token Exchange como bala de plata para todo problema de propagación de identidad. No lo es: cada intercambio añade latencia, complica el plano de control y mueve trabajo al AS que hay que dimensionar. En muchas arquitecturas, un *service mesh* con mTLS y SPIFFE propagando contexto por cabeceras firmadas es más eficiente para llamadas internas sin cambio de *audience*.

El segundo es no validar las relaciones de confianza. Token Exchange solo es seguro si el AS aplica políticas estrictas sobre quién puede intercambiar qué. Permitir que cualquier cliente obtenga un token derivado con cualquier *audience* reabre el agujero que RFC 8693 venía a cerrar. La configuración correcta define explícitamente qué cliente puede actuar como delegado, para qué *audiences* y con qué *scopes*.

El tercero es confundir `act` con `azp`. El *claim* `azp` (*authorized party*, OIDC) identifica al cliente al que se emitió el token originalmente; `act` identifica al actor intermedio en una delegación. Escribir políticas basadas en el *claim* equivocado produce decisiones de autorización incorrectas que rara vez saltan en pruebas.

> Si tu configuración de Token Exchange permite que cualquier servicio intercambie cualquier token, no has desplegado Token Exchange, has desplegado un reemisor abierto.
{: .prompt-warning }

## Alternativas y cuándo preferirlas

Token Exchange no es la única respuesta. JAR (JWT-Secured Authorization Request, RFC 9101, 2020) y las *assertion-based flows* de RFC 7521 y RFC 7523 cubren casos donde el cliente ya dispone de una aserción firmada (por ejemplo, un certificado federado) y quiere obtener un *access token* sin pasar por flujo interactivo. Son complementarias, no sustitutas.

Para cadenas de servicios dentro del mismo dominio de confianza, la propagación de contexto por cabeceras firmadas dentro de un *service mesh* con mTLS y SPIFFE resuelve el problema con menos infraestructura, asumiendo que el mesh garantiza la autenticidad de las cabeceras. Cuando la llamada cruza dominios o cuando la política de autorización necesita un token criptográficamente verificable de extremo a extremo, Token Exchange es la opción correcta.

## Artículos relacionados en esta serie

- [OAuth 2.0](/posts/OAuth2/)
- [JSON Web Token](/posts/JWT/)
- [OpenID Connect](/posts/OIDC/)
- [Keycloak, Okta y los IdP modernos](/posts/Keycloak-Okta-IdP/)
- [El futuro de OAuth](/posts/OAuth-Futuro/)

## Referencias

- RFC 7519 (mayo 2015). *JSON Web Token (JWT)*.
- RFC 7521 (mayo 2015). *Assertion Framework for OAuth 2.0 Client Authentication and Authorization Grants*.
- RFC 7523 (mayo 2015). *JSON Web Token (JWT) Profile for OAuth 2.0 Client Authentication and Authorization Grants*.
- RFC 8693 (enero 2020). *OAuth 2.0 Token Exchange*.
- RFC 9101 (agosto 2021). *The OAuth 2.0 Authorization Framework: JWT-Secured Authorization Request (JAR)*.
- Keycloak. *Token Exchange Documentation*.
- Microsoft. *Identity Platform and OAuth 2.0 On-Behalf-Of Flow*.
