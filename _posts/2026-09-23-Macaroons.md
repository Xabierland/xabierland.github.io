---
title: "Macaroons: la alternativa olvidada a JWT para delegación descentralizada"
author: Xabierland
description: >-
    Los macaroons son credenciales bearer construidas sobre HMAC encadenado que permiten atenuación descentralizada mediante caveats. Resuelven el problema de delegar permisos restringidos sin contactar al emisor, algo que JWT no puede hacer de forma nativa. A pesar de su elegancia técnica, el ecosistema OAuth y JWT los dejó en un nicho del que nunca salieron del todo.
date: 2026-09-23 10:00
categories: [DevSecOps, Seguridad de Aplicaciones]
tags: [Macaroons, JWT, Capability, Autorización, Delegación, HMAC, OAuth]
---

Este artículo forma parte de la serie [La evolución de la seguridad en aplicaciones modernas](/posts/La-evolucion-de-la-seguridad-en-aplicaciones-modernas/). Hablar de tokens hoy es hablar casi exclusivamente de JWT: inmutables, autocontenidos y firmados. Pero en 2014, un equipo de Google propuso un diseño radicalmente distinto que nunca llegó al mainstream y que sigue siendo la respuesta más elegante a una pregunta incómoda: ¿cómo restrinjo un permiso existente sin pedir un token nuevo?

## Por qué los macaroons existen

El paper original, *Macaroons: Cookies with Contextual Caveats for Decentralized Authorization in the Cloud* (Birgisson, Politz, Erlingsson, Taly, Vrable, Lentczner, NDSS 2014), partió de una frustración concreta. En una arquitectura de servicios con millones de objetos, un usuario que quiere compartir un archivo con un tercero no debería tener que volver al servidor de autorización para pedir un token con alcance reducido. El usuario ya tiene el permiso; debería poder derivar una versión atenuada por sí mismo.

JWT no permite esto. Un JWT es inmutable: cualquier cambio rompe la firma, y solo el emisor puede firmar. Esto convierte al AS (Authorization Server) en un cuello de botella para cualquier operación de delegación fina. Los macaroons invierten el modelo. Un macaroon es una credencial bearer donde cualquier portador puede añadir restricciones (*caveats*) sin hablar con el emisor, pero nadie puede quitarlas. La verificación sigue siendo criptográficamente segura porque cada caveat se incorpora encadenando HMACs.

## Cómo funcionan por dentro

Un macaroon arranca con una clave secreta que conoce el emisor. Sobre esa clave se calcula un HMAC de un identificador inicial (por ejemplo, el ID del recurso y el usuario). Ese HMAC se convierte en la "firma actual" del macaroon. Cada vez que alguien añade un caveat, se recalcula un nuevo HMAC usando la firma anterior como clave y el texto del caveat como mensaje. El resultado sustituye a la firma anterior.

Este encadenamiento tiene una propiedad crítica: solo quien conoce la firma actual puede derivar la siguiente. Y como la firma actual depende de todos los caveats previos, un atacante no puede quitar ni modificar un caveat sin invalidar la cadena. El verificador, que sí conoce la clave raíz, reconstruye el HMAC siguiendo la cadena de caveats y compara.

Hay dos tipos de caveats. Los *first-party caveats* son condiciones que el verificador puede evaluar localmente: "expira en 5 minutos", "solo lectura", "path prefix /users/42". Los *third-party caveats* delegan la verificación a otro servicio: "el usuario debe haber hecho MFA en el IdP corporativo". Para resolver un third-party caveat, el cliente obtiene un *discharge macaroon* emitido por el tercero que demuestra el cumplimiento; los dos macaroons se presentan juntos en la petición.

## Attenuation sin contactar al emisor

El caso canónico es compartir. Alicia tiene un macaroon que le da acceso completo a su cubo de almacenamiento. Quiere enviarle a Bob un enlace que le permita leer solo el directorio `/fotos/2026/` durante una hora. Con JWT tendría que ir al AS, autenticarse, pedir un token con ese scope específico y expiración corta, y pasárselo a Bob. Con macaroons, Alicia añade dos caveats al macaroon que ya tiene (`path = /fotos/2026/*` y `expires = now+3600`) y le envía el resultado a Bob. Bob lo usa; el servidor verifica la cadena HMAC y evalúa los caveats.

No ha habido ninguna interacción con el emisor. Esto es una capability en el sentido clásico: la posesión del token es la autoridad, y la autoridad se puede restringir localmente.

> La atenuación es unidireccional. Un portador puede añadir restricciones pero nunca relajarlas. Esto es lo que hace que el modelo sea seguro incluso cuando cualquiera puede reescribir el macaroon.
{: .prompt-info }

## Verificación offline

La segunda ventaja práctica es que la verificación es completamente local, sin llamadas de red. A diferencia de la introspección de tokens opacos definida en `RFC 7662 (OAuth 2.0 Token Introspection)`, el verificador solo necesita la clave raíz y la lógica para evaluar los caveats que entiende. Esto los hace atractivos en entornos donde la latencia importa: storage, edge, sistemas distribuidos con replicación.

Comparado con JWT, la diferencia está en la semántica, no en el coste de verificación (ambos son operaciones locales). La cuestión es qué puedes expresar. Un JWT dice "el emisor afirma X". Un macaroon dice "el emisor autoriza Y sujeto a estas condiciones que cualquier portador intermedio ha podido endurecer".

## Dónde se usan en producción

Google Cloud Storage los utiliza internamente para firmas de URL con delegación. Tailscale construye sus *machine keys* con un esquema inspirado en macaroons para que los nodos puedan derivar credenciales restringidas sin volver al plano de control. Cloudflare Hyperdrive y algunas piezas de su stack de acceso también los emplean. En el mundo open source, `libmacaroons` y `pymacaroons` son las implementaciones de referencia, y el protocolo Lightning Network de Bitcoin los usa para autorizar pagos condicionales.

El patrón común en todos estos casos es el mismo: escala masiva, necesidad de delegación fina, y aversión a que cada operación de autorización pase por un punto central.

## Cuándo sí tiene sentido adoptarlos

Los macaroons encajan cuando el problema tiene al menos dos de estas características. Primero, delegación descentralizada: los portadores necesitan restringir sus propios permisos para pasárselos a otros sin round-trip al AS. Segundo, capability-based security: el modelo mental del dominio es "poseer el token es tener el derecho", más que "el token afirma una identidad que se evalúa contra políticas externas". Tercero, latencia crítica y volumen alto de verificaciones: la red al AS es un lujo que no te puedes permitir.

Storage, CDNs, sistemas de compartición de archivos, y APIs internas entre microservicios que implementan capability patterns son los nichos naturales.

## Cuándo no

Si tu problema es *identity federation* entre organizaciones, OIDC (OpenID Connect) y JWT resuelven mejor el caso. Los macaroons no llevan identidad del emisor firmada de forma estándar; llevan capacidades. Si necesitas que un auditor lea el token y sepa "este usuario, autenticado por este IdP, con estos claims", JWT es la respuesta.

Si tu equipo no va a invertir en entender el modelo, tampoco. Los macaroons son conceptualmente más complejos que JWT, y la documentación pública es escasa. Debugar un caveat mal evaluado o una cadena rota requiere herramientas que no existen con la misma madurez que el ecosistema JWT.

Y si operas en un entorno muy regulado donde los auditores esperan ver terminología OAuth/OIDC estándar, introducir macaroons añade fricción de cumplimiento sin aportar ventajas proporcionales.

> No uses macaroons para identidad. Úsalos para autorización delegada. Son capabilities, no assertions.
{: .prompt-warning }

## Por qué no ganaron

La pregunta honesta es por qué un diseño técnicamente superior en su nicho no desplazó a JWT. La respuesta tiene tres partes.

La primera es inercia del ecosistema. Cuando el paper salió en 2014, OAuth 2.0 llevaba tres años como estándar y JWT estaba a punto de consolidarse con `RFC 7519 (JSON Web Token)` en mayo de 2015. Todo el tooling, todas las librerías, todos los IdPs apuntaban ya en esa dirección. Cambiar de rumbo requería un coste colectivo que nadie tenía incentivos para pagar.

La segunda es la falta de estandarización IETF. Los macaroons nunca tuvieron un RFC. Cada implementación (libmacaroons, pymacaroons, macaroons.io) tiene pequeñas variaciones en serialización, formato de caveats y convenciones. Sin un estándar común, la interoperabilidad entre stacks distintos es frágil.

La tercera es que el problema que resuelven elegantemente (atenuación descentralizada) resultó ser menos común de lo que parecía. La mayoría de las arquitecturas corporativas prefieren centralizar las decisiones de autorización en un PDP (Policy Decision Point) o en un AS, aunque eso introduzca latencia, porque simplifica la auditoría y el cumplimiento normativo. La descentralización es técnicamente más potente pero políticamente más incómoda.

## Encaje arquitectónico actual

En un stack moderno, los macaroons pueden coexistir con JWT y OAuth 2.0 si separas capas. Un usuario se autentica contra el IdP corporativo y obtiene un JWT que demuestra identidad. A partir de ahí, si necesita delegar capacidades finas sobre recursos específicos (un URL firmado para un objeto de storage, una credencial compartible con atenuación), el servicio emite un macaroon a partir del JWT. La identidad queda federada y auditada arriba; la capacidad se distribuye y restringe abajo.

Este patrón es el que usan varios sistemas en producción, aunque rara vez lo documenten con esa claridad. Es también la única forma realista de introducir macaroons en una organización que no quiere romper su inversión en OAuth y OIDC.

## Artículos relacionados en esta serie

- [JSON Web Token](/posts/JWT/)
- [OAuth 2.0](/posts/OAuth2/)
- [Token Exchange](/posts/Token-Exchange/)
- [Gestión de sesiones y revocación](/posts/Sesiones-y-Revocacion/)
- [El futuro de OAuth](/posts/OAuth-Futuro/)

## Referencias

- Birgisson, A., Politz, J. G., Erlingsson, Ú., Taly, A., Vrable, M., Lentczner, M. (2014). *Macaroons: Cookies with Contextual Caveats for Decentralized Authorization in the Cloud*. NDSS Symposium 2014, Google.
- RFC 7519 (mayo 2015). *JSON Web Token (JWT)*.
- RFC 7662 (octubre 2015). *OAuth 2.0 Token Introspection*.
- Google (2014). *libmacaroons: C library for Macaroons*.
- Tailscale (2023). *How Tailscale Works: Machine Keys and Node Authentication*.
