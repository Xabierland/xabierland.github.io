---
title: "El futuro de OAuth: 2.1, DPoP, PAR, RAR y la promesa de GNAP"
author: Xabierland
description: >-
    OAuth 2.0 cumplió más de una década acumulando parches, extensiones y lecciones dolorosas. La respuesta del IETF es doble: OAuth 2.1 consolida las buenas prácticas en un único documento, mientras DPoP, PAR y RAR cubren los agujeros del modelo original. En paralelo, GNAP se perfila como la siguiente generación sin buscar compatibilidad.
date: 2026-09-09 10:00
categories: [DevSecOps, Seguridad de Aplicaciones]
tags: [OAuth2.1, DPoP, PAR, RAR, GNAP, Tokens, Estándares]
---

OAuth 2.0 no es un protocolo, es una familia. RFC 6749 (2012) definió un marco flexible donde cabían múltiples *grant types*, múltiples formas de presentar credenciales y múltiples interpretaciones de qué significa "seguro". Una década después, la familia tenía más excepciones que reglas. Este artículo forma parte de la serie [La evolución de la seguridad en aplicaciones modernas](/posts/La-evolucion-de-la-seguridad-en-aplicaciones-modernas/).

## Por qué OAuth 2.0 necesitaba una limpieza

El problema no fue el diseño original, fue la deriva. A medida que OAuth se adoptaba en contextos para los que no fue pensado (aplicaciones móviles, SPAs, APIs financieras, IoT), cada caso abría nuevas vulnerabilidades. La respuesta del grupo de trabajo OAuth del IETF fue publicar el *OAuth 2.0 Security Best Current Practice* (RFC 9700, 2024), una lista creciente de qué hacer y qué evitar. Ese documento cumplió su función durante años, pero su propio tamaño lo hacía impracticable como guía para implementadores nuevos.

De ahí la decisión de consolidar en OAuth 2.1 las recomendaciones que ya eran consenso, eliminando los *grant types* obsoletos y elevando a requisito lo que antes era opcional. Paralelamente, nuevas especificaciones cerraron agujeros concretos que el diseño original dejaba abiertos.

## OAuth 2.1: la consolidación

OAuth 2.1, en estado de *Internet-Draft* avanzado (draft-ietf-oauth-v2-1), no añade funcionalidad nueva. Normaliza lo que la comunidad ya consideraba obligatorio. Los cambios sustantivos son cinco.

PKCE (Proof Key for Code Exchange, RFC 7636, 2015) pasa a ser obligatorio para todos los clientes que usen *authorization code flow*, no solo los públicos. El razonamiento es que los *authorization code injection attacks* afectan también a clientes confidenciales en ciertas condiciones, y el coste de exigir PKCE siempre es mínimo.

El *Implicit grant* desaparece. Durante años fue el flujo recomendado para SPAs porque los navegadores no podían guardar un *client secret*. La aparición de PKCE resolvió ese problema y el *Implicit grant*, que emitía *access tokens* directamente en la URL, quedó como vector de exfiltración por *browser history*, *referrers* y *logs*.

El *Resource Owner Password Credentials grant* (ROPC) desaparece también. Pedir usuario y contraseña directamente al cliente contradice el propósito original de OAuth: delegar la autenticación al *Authorization Server*. ROPC sobrevivía como muleta para migraciones desde autenticación básica, pero su permanencia era incompatible con federación, MFA y políticas de acceso modernas.

La validación de *redirect URI* pasa a ser estricta: coincidencia exacta, sin *wildcards*, sin comparación *case-insensitive*. Muchas vulnerabilidades históricas aprovecharon *matches* laxos en esta validación para redirigir *authorization codes* a dominios controlados por el atacante.

Los *refresh tokens* pasan a requerir una de dos protecciones: rotación (emitir un nuevo *refresh token* con cada uso e invalidar el anterior) o estar vinculados al emisor (*sender-constrained*, por ejemplo con DPoP o mTLS). La lógica es que un *refresh token* estático en manos de un atacante otorga acceso indefinido al recurso.

## DPoP: tokens atados al portador legítimo

DPoP (*Demonstrating Proof of Possession*, RFC 9449, septiembre de 2023) ataca uno de los problemas más persistentes del OAuth original: los *bearer tokens* son válidos para cualquiera que los porte. Si un atacante roba un *access token* mediante XSS, extensión maliciosa o intercepción, puede usarlo contra el recurso sin ninguna otra credencial.

La idea de DPoP es vincular criptográficamente el *token* a una clave que solo el cliente legítimo conoce. Cuando el cliente solicita un *token*, genera un par de claves efímero y envía la clave pública al *Authorization Server* en una cabecera `DPoP`. El AS incluye en el *token* emitido una *confirmation claim* (`cnf`) con el *hash* de esa clave pública. Cada vez que el cliente usa el *token* contra un recurso, adjunta una nueva prueba DPoP: un JWT firmado con la clave privada, que incluye el método HTTP, la URI y una marca de tiempo. El recurso verifica la firma, comprueba la vinculación con el *token* y rechaza la petición si no cuadra.

El resultado es que un *token* robado sin la clave privada asociada no sirve. La diferencia con mTLS-bound tokens (RFC 8705, 2020) es operativa: DPoP no requiere que el cliente tenga un certificado X.509 válido ni que la infraestructura soporte mTLS de extremo a extremo. Para SPAs y clientes móviles, donde mTLS es impracticable, DPoP cierra el agujero sin infraestructura adicional.

> DPoP no elimina el riesgo del token robado, lo hace irrelevante. Sin la clave privada efímera, el token es un pedazo de texto que no autoriza nada.
{: .prompt-info }

## PAR: la petición de autorización ya no viaja por el canal del navegador

PAR (*Pushed Authorization Requests*, RFC 9126, septiembre de 2021) resuelve el problema de que los parámetros de la *authorization request* viajan tradicionalmente por el *front-channel*: el navegador del usuario los lleva como query string hasta el *Authorization Server*. Eso los expone a manipulación por extensiones, *malware* y técnicas de *URL tampering*.

PAR propone una inversión simple. Antes de redirigir al usuario, el cliente envía los parámetros de la petición al AS por *back-channel* (HTTPS directo, autenticado como cliente), recibe un identificador corto (*request_uri*) y redirige al usuario con solo ese identificador. El AS recupera los parámetros originales a partir del identificador, imposibles de manipular desde el navegador.

Esto no solo cierra vectores de *tampering*; también habilita peticiones complejas que no cabrían en una URL (por ejemplo, *authorization_details* detalladas o *claims requests* extensas). FAPI 2.0, el perfil financiero del OpenID Foundation, exige PAR para todos los clientes.

## RAR: autorización estructurada

RAR (*Rich Authorization Requests*, RFC 9396, mayo de 2023) ataca la limitación más visible del OAuth original: los *scopes* son cadenas planas. Un *scope* `payment` no puede expresar "autorizar un pago de hasta mil euros al beneficiario X desde la cuenta Y". O el cliente crea decenas de *scopes* sintéticos, o pide un *scope* amplio y deja los detalles a la aplicación, perdiendo control fino.

RAR introduce un nuevo parámetro `authorization_details` que acepta JSON estructurado. Cada entrada tiene un `type` y atributos específicos del tipo. Los emisores pueden registrar tipos (p. ej., `payment_initiation`, `account_information`) con su esquema, y el AS valida que la petición encaja en los permisos del usuario y el cliente. El *token* emitido incluye los *authorization_details* aceptados, y el recurso los inspecciona al servir la operación.

Para dominios regulados (banca abierta, sanidad, seguros), RAR es lo que faltaba para expresar autorización con la granularidad que la regulación exige, sin recurrir a extensiones propietarias.

## FAPI 2.0: ensamblaje para sectores críticos

FAPI 2.0 (Financial-grade API Security Profile 2.0) del OpenID Foundation, en *Final* desde 2024, es un perfil que combina OAuth 2.1, PAR, DPoP o mTLS y RAR opcional en un conjunto coherente y certificable. No es un estándar nuevo, es la instrucción de cómo montar los anteriores sin dejarse nada. Quien implementa FAPI 2.0 obtiene un sistema resistente a las clases de ataques conocidas contra OAuth, a cambio de asumir complejidad.

Aunque nació para APIs financieras (PSD2 en Europa, Open Banking en Reino Unido), FAPI 2.0 se usa fuera del sector cuando los requisitos de seguridad lo justifican.

## GNAP: la siguiente generación

GNAP (*Grant Negotiation and Authorization Protocol*, RFC 9635, octubre de 2024) es el trabajo del IETF para redefinir la autorización delegada desde cero. No intenta ser *OAuth 3*; su estructura de mensajes y su modelo conceptual son distintos. Hereda de XYZ, una propuesta de Justin Richer que precede a la adopción en el IETF.

Las diferencias clave con OAuth son cuatro. La petición es iniciada por el cliente con un único mensaje al AS que describe qué acceso quiere, quién lo quiere y con qué credenciales de cliente, en lugar de distribuir la información entre endpoints distintos. Es multi-ronda: el AS puede pedir al cliente información adicional, factores MFA o interacción del usuario de forma explícita y estructurada. Soporta múltiples sujetos en la misma petición (un cliente puede pedir acceso a recursos de varios usuarios a la vez). Y las credenciales de cliente están desacopladas del mecanismo (llaves, certificados, HTTP signatures).

El coste de adopción es alto precisamente porque no hay compatibilidad hacia atrás. GNAP no es un camino de actualización desde OAuth; es un protocolo paralelo. Su adopción será gradual, probablemente primero en *greenfields* críticos (banca, sanidad, identidad soberana) y solo más tarde en uso general.

> GNAP no va a reemplazar a OAuth en cinco años. La pregunta correcta no es si migrar, sino si tu próximo proyecto greenfield debería nacer en GNAP o esperar.
{: .prompt-tip }

## Recomendaciones hoy

Para un arquitecto que toma decisiones en 2026, las prioridades son razonablemente claras. PKCE en todos los clientes, sin excepciones. Eliminación de *Implicit* y ROPC de cualquier código que aún los use. Validación estricta de *redirect URI*. Rotación obligatoria de *refresh tokens*. Estas son las mejoras de OAuth 2.1 que no justifican esperar al RFC final.

DPoP para clientes públicos (SPAs, móviles) cuyo token pueda ser robado por XSS o extensiones; mTLS para backends que ya tengan PKI interna. PAR cuando el AS lo soporte y especialmente si se manejan *authorization_details* complejas. RAR cuando la autorización necesita granularidad fina (montos, beneficiarios, recursos específicos) y el sector lo exija o lo aconseje.

GNAP como tema de vigilancia, con pruebas de concepto cuando aparezcan casos nuevos sin compromisos con OAuth, especialmente donde el modelo multi-sujeto o multi-ronda aporte valor real.

## Consideraciones de migración

Pocos incidentes son tan dolorosos como una migración mal planificada del mecanismo de autenticación. Tres principios ayudan a reducir el riesgo. Primero, introducir las nuevas características como opciones antes de hacerlas obligatorias, permitiendo que los clientes migren a su ritmo. Segundo, instrumentar telemetría por versión de flujo para saber qué clientes siguen en el modo antiguo antes de retirarlo. Tercero, asumir que los *tokens* en circulación sobreviven a cualquier cambio: planificar periodos de transición donde el AS acepta ambos modelos hasta que los *tokens* antiguos caduquen por su propio TTL (*Time To Live*).

La buena noticia es que OAuth 2.1, DPoP, PAR y RAR son aditivos. No rompen clientes existentes; los dejan atrás. La migración se hace cliente a cliente, no de golpe.

## Artículos relacionados en esta serie

- [OAuth 2.0](/posts/OAuth2/)
- [PKCE](/posts/PKCE/)
- [JSON Web Token](/posts/JWT/)
- [Token Exchange](/posts/Token-Exchange/)
- [mTLS](/posts/mTLS/)

## Referencias

- RFC 6749 (octubre 2012). *The OAuth 2.0 Authorization Framework*.
- RFC 7636 (septiembre 2015). *Proof Key for Code Exchange by OAuth Public Clients*.
- RFC 8705 (febrero 2020). *OAuth 2.0 Mutual-TLS Client Authentication and Certificate-Bound Access Tokens*.
- RFC 9126 (septiembre 2021). *OAuth 2.0 Pushed Authorization Requests (PAR)*.
- RFC 9396 (mayo 2023). *OAuth 2.0 Rich Authorization Requests (RAR)*.
- RFC 9449 (septiembre 2023). *OAuth 2.0 Demonstrating Proof of Possession (DPoP)*.
- RFC 9635 (octubre 2024). *Grant Negotiation and Authorization Protocol (GNAP)*.
- RFC 9700 (2024). *Best Current Practice for OAuth 2.0 Security*.
- IETF OAuth Working Group. *draft-ietf-oauth-v2-1: OAuth 2.1*.
- OpenID Foundation (2024). *FAPI 2.0 Security Profile*.
