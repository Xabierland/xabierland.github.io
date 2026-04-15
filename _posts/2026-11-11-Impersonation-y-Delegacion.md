---
title: "Impersonation vs delegación: la semántica que rompe auditorías"
author: Xabierland
description: >-
    Impersonation y delegación parecen sinónimos pero modelan cadenas de actores radicalmente distintas. La primera borra al intermediario de los logs; la segunda lo preserva. Entender la diferencia es crítico para cumplir auditorías de trazabilidad en entornos regulados como SOX, GDPR o HIPAA.
date: 2026-11-11 10:00
categories: [DevSecOps, Seguridad de Aplicaciones]
tags: [Impersonation, Delegación, Token Exchange, OAuth, Auditoría, RFC 8693, OBO]
---

Este artículo forma parte de la serie [La evolución de la seguridad en aplicaciones modernas](/posts/La-evolucion-de-la-seguridad-en-aplicaciones-modernas/). Hay una pregunta que aparece en toda auditoría seria y que separa las arquitecturas maduras de las que solo parecen serlo: cuando un sistema ejecuta una acción en nombre de un usuario, ¿quién figura en los logs downstream? La respuesta depende de si has diseñado impersonation o delegación, y confundirlas es más común de lo que debería.

## Dos modelos con un mismo efecto aparente

En la superficie, ambos patrones producen el mismo resultado: un servicio B llama a un recurso R usando un token que representa al usuario U. La diferencia está en cómo se codifica la cadena de actores.

En impersonation, B obtiene un token donde el `sub` (subject) es U y no hay ninguna referencia a B. Para R, la petición viene del usuario U. Los logs de R dicen "U hizo esta operación", y punto. B ha desaparecido de la cadena de auditoría.

En delegación, B obtiene un token donde el `sub` sigue siendo U pero el token incluye un claim `act` (actor) que describe quién está actuando en nombre de U. Para R, la petición viene del usuario U ejecutada por el servicio B. Los logs pueden reflejar ambos actores.

## RFC 8693 y el claim act

`RFC 8693 (OAuth 2.0 Token Exchange)`, de enero de 2020, estandarizó el mecanismo para obtener tokens que representen estas semánticas. Define el endpoint de token exchange donde un cliente presenta un `subject_token` (el token del usuario) y opcionalmente un `actor_token` (el token del servicio que actúa), junto con un parámetro `requested_token_type`.

El claim clave es `act`, registrado en la JWT Claims Registry. Su estructura es un objeto JSON que contiene al menos un `sub` identificando al actor, y puede anidarse: `act.act` representa al actor del actor, construyendo una cadena de delegación transitiva cuando un servicio B delega a su vez en un servicio C.

El RFC distingue explícitamente los dos casos. Con `actor_token` presente y el AS emitiendo un token con `act`, tienes delegación. Sin `actor_token`, o con una configuración donde el AS emite un token sin `act` apuntando al cliente, tienes impersonation. Ambos son válidos según el estándar; la decisión es de política.

## Por qué la diferencia importa para auditoría

Las normativas de cumplimiento se construyen sobre una premisa: todo evento debe ser atribuible a un actor identificable. SOX exige trazabilidad de operaciones financieras hasta el responsable humano. GDPR, en su artículo 30, obliga a mantener un registro de actividades de tratamiento con información suficiente para reconstruir quién accedió a qué datos personales. HIPAA, en su Security Rule, requiere audit controls que registren y examinen la actividad en sistemas con PHI (Protected Health Information).

En todos estos marcos, un log que dice "U ejecutó la operación" cuando en realidad fue un servicio automático actuando con credenciales de U es, técnicamente, inexacto. Si un auditor reconstruye la cadena y descubre que el evento no pudo venir directamente de U (porque U estaba fuera de horario, o porque el patrón de la operación es claramente automatizado), surge una pregunta difícil: ¿el sistema oculta deliberadamente al verdadero actor?

La delegación explícita resuelve esto. El log contiene tanto U como B, el auditor puede rastrear el flujo completo, y la responsabilidad queda clara: U autorizó a B a actuar, B ejecutó la acción concreta.

## Un ejemplo que aclara la diferencia

Considera un workflow de aprobación de facturas. El usuario U configura una regla: "aprueba automáticamente facturas de este proveedor por debajo de 500 euros". Un servicio automatizado B evalúa las facturas entrantes y ejecuta las aprobaciones que cumplen la regla.

Con impersonation, cada aprobación aparece en los logs como "aprobada por U". Un auditor que investigue un fraude con facturas duplicadas verá que U aprobó muchas facturas a alta velocidad, lo cual es imposible para un humano, pero el log no le dirá que fue un automatismo. Si U niega haber aprobado esas facturas, la reconstrucción forense es costosa.

Con delegación, cada aprobación aparece como "aprobada por U, ejecutada por workflow-service". El auditor distingue inmediatamente las aprobaciones manuales de U de las automatizadas por B. Puede verificar la configuración de la regla en el momento de cada aprobación y reconstruir la intención original de U.

> Impersonation hace invisible al actor real. En operaciones con impacto regulatorio, esa invisibilidad es una deuda de cumplimiento esperando ser cobrada.
{: .prompt-warning }

## On-Behalf-Of de Microsoft

Microsoft documenta desde hace años un patrón llamado On-Behalf-Of (OBO), implementado en Entra ID. Es una variante de delegación que precede a RFC 8693 pero converge en espíritu.

En OBO, una API A recibe un token del usuario U, y para llamar a una API downstream B necesita un token válido hacia B. En lugar de pedir un token nuevo con las credenciales de A (que sería impersonation del servicio sobre U) o simplemente reenviar el token de U (que requeriría que U tuviera permisos explícitos sobre B), A llama al endpoint de token exchange de Entra ID presentando el token de U y pide un token hacia B.

El token resultante mantiene a U como subject pero el AS registra que A participó en la cadena. Entra ID puede emitir tokens con claim `xms_act` o equivalentes que preservan la información del actor intermedio. OBO es la implementación canónica de delegación en el ecosistema Microsoft.

## Cuándo impersonation es legítimo

Hay un caso donde impersonation está justificada y es el soporte técnico con break-glass. Un ingeniero de soporte necesita reproducir un bug que solo ocurre en la cuenta de un cliente concreto. Pedirle al cliente que comparta su contraseña es impensable. Darle al ingeniero un token que le permita ver la aplicación exactamente como la ve el cliente, durante un periodo limitado, es razonable.

Lo que hace este caso aceptable no es el mecanismo técnico; es el sistema de controles alrededor. Impersonation legítima requiere al menos tres elementos. Aprobación explícita y registrada fuera del sistema impersonado (un ticket, un workflow de break-glass). Ventana temporal limitada con expiración forzada. Auditoría externa e inmutable que registre quién hizo impersonation, cuándo, con qué justificación y qué acciones ejecutó. El log de la aplicación verá las acciones como del cliente; el log externo del sistema de soporte verá al ingeniero.

Este patrón de "impersonation con trazabilidad fuera de banda" es común en soporte de primer nivel de grandes SaaS. Lo problemático no es la impersonation en sí, sino hacerla sin el andamiaje de auditoría externa.

## Dónde se cruza con PAM

El patrón anterior conecta con Privileged Access Management. Un sistema PAM no solo custodia credenciales privilegiadas; también graba sesiones, exige aprobación previa, y expone una auditoría independiente del sistema accedido. Cuando impersonation ocurre a través de un PAM, la trazabilidad se reconstruye desde los logs del PAM, que no dependen de que la aplicación impersonada registre al actor real.

En organizaciones maduras, impersonation sin PAM es sospechosa por defecto. Delegación con claim `act` puede coexistir con logs aplicativos normales. Impersonation necesita infraestructura adicional.

## Checklist para el diseño

Hay una pregunta operativa que resume el trade-off. Si un regulador o un cliente aparece mañana y te pregunta "¿quién ejecutó esta acción concreta, en este momento concreto, sobre este dato concreto?", ¿puedes responder con certeza y soporte documental?

Si la respuesta depende de correlacionar logs de varios sistemas, reconstruir estados del directorio y hacer arqueología forense, algo está mal diseñado. Si la respuesta sale de una consulta al log del sistema afectado, con el actor humano y el actor técnico ambos presentes, el diseño es sólido.

> Diseña asumiendo que tu sistema de autorización será auditado. Delegación con `act` por defecto; impersonation solo con break-glass documentado y auditoría externa.
{: .prompt-tip }

## Cuándo elegir cada uno

Elige delegación cuando la acción tenga impacto regulatorio, financiero o sobre datos personales. Cuando existan cadenas de servicios automatizados actuando en nombre de usuarios. Cuando quieras que los logs de la aplicación sean autosuficientes para responder preguntas de auditoría. Este es el default razonable en cualquier arquitectura moderna.

Elige impersonation solo cuando la experiencia del consumidor downstream requiera ver al usuario original sin ambigüedad (el caso de soporte técnico), y solo si tienes el andamiaje de control externo (PAM, aprobación fuera de banda, auditoría inmutable). No elijas impersonation por simplicidad; el coste de cumplimiento llega más tarde y es más caro.

## Cuándo no aplicar ninguno

Si el servicio que llama al recurso no actúa en nombre de un usuario específico sino en nombre de sí mismo (un batch nocturno, un sistema de métricas, un health check), no uses ni impersonation ni delegación. Usa client credentials con una identidad propia del servicio. Intentar forzar un modelo user-centric en operaciones que no lo son ensucia los logs y difumina responsabilidades.

## Encaje arquitectónico

El encaje canónico combina varias piezas. El IdP emite tokens de usuario con OIDC. Los servicios intermedios hacen token exchange vía RFC 8693 cuando necesitan actuar en cadenas multi-servicio, preservando el claim `act`. Los servicios que operan sin usuario usan client credentials con identidad propia. El sistema de logging centralizado extrae tanto `sub` como `act` de los tokens y los registra explícitamente, no como un blob indistinto.

Esta disciplina pequeña, aplicada consistentemente desde el principio, es lo que diferencia una auditoría que se resuelve en horas de una que se resuelve en semanas.

## Artículos relacionados en esta serie

- [Token Exchange](/posts/Token-Exchange/)
- [OAuth 2.0](/posts/OAuth2/)
- [OpenID Connect](/posts/OIDC/)
- [Privileged Access Management](/posts/Privileged-Access-Management/)
- [Gestión de sesiones y revocación](/posts/Sesiones-y-Revocacion/)

## Referencias

- RFC 8693 (enero 2020). *OAuth 2.0 Token Exchange*.
- RFC 7519 (mayo 2015). *JSON Web Token (JWT)*.
- Microsoft (2024). *Microsoft identity platform and OAuth 2.0 On-Behalf-Of flow*.
- IAPP (2023). *GDPR Article 30: Records of Processing Activities*.
- HHS (2013). *HIPAA Security Rule: Audit Controls 45 CFR 164.312(b)*.
- IETF (2020). *JWT Claims Registry: act claim*.
