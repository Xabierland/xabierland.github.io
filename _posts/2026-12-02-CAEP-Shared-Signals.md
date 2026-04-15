---
title: "CAEP y Shared Signals Framework: señales continuas de identidad"
author: Xabierland
description: >-
    El Shared Signals Framework de la OpenID Foundation define cómo IdPs, aplicaciones y plataformas intercambian eventos de seguridad en tiempo real. CAEP es el perfil que cubre sesiones, tokens y cambios en la postura de identidad. Es la pieza que convierte la autenticación continua de Zero Trust en algo operable más allá de una diapositiva.
date: 2026-12-02 10:00
categories: [DevSecOps, Seguridad de Aplicaciones]
tags: [CAEP, SSF, OpenID, ZeroTrust, SET, Identidad, Eventos]
---

Este artículo forma parte de la serie [La evolución de la seguridad en aplicaciones modernas](/posts/La-evolucion-de-la-seguridad-en-aplicaciones-modernas/).

El modelo clásico de autenticación federada tiene un hueco incómodo: una vez emitido un token con caducidad de, digamos, una hora, el proveedor de identidad pierde el control sobre él. Si el usuario es despedido a los cinco minutos de loguearse, o su dispositivo se detecta comprometido, o un administrador revoca la cuenta, el token sigue siendo válido para cualquier aplicación que lo acepte hasta que caduque o hasta que cada aplicación, por su cuenta, pregunte al IdP si sigue vivo. CAEP y el Shared Signals Framework llenan ese hueco con un canal de eventos push estandarizado.

## Por qué nace Shared Signals

El problema se manifiesta en cuanto una organización tiene más de un sistema de identidad. Google lo vio pronto con su Cross-Account Protection, lanzado en 2017 para notificar a aplicaciones de terceros cuando una cuenta Google se comprometía. Microsoft publicó en 2022 Continuous Access Evaluation en Entra ID para resolver el mismo problema entre sus propios servicios. Okta se sumó en 2023 con soporte nativo. Cada uno tenía una solución propietaria, y la necesidad de un estándar era evidente.

La OpenID Foundation formalizó el Shared Signals Framework (SSF) como marco común, con dos perfiles principales: CAEP (Continuous Access Evaluation Profile) para eventos relacionados con sesiones activas y postura de acceso, y RISC (Risk and Incident Sharing and Coordination) para eventos de compromiso de cuenta más orientados a ecosistemas B2C. Ambos comparten transporte, formato y modelo de suscripción; se diferencian en qué eventos describen.

El valor frente a polling es directo. Un SP (Service Provider) que quiera validar una sesión cada minuto contra el IdP genera un coste N×M que no escala. Un modelo push donde el IdP solo envía eventos cuando algo cambia reduce el tráfico a lo estrictamente necesario, y cierra la ventana de exposición de minutos a segundos.

## Arquitectura: Transmitter y Receiver

El framework modela dos roles. El Transmitter es la entidad que genera eventos (típicamente el IdP o una plataforma de postura de dispositivo). El Receiver es la entidad que los consume (una aplicación, una API gateway, un SIEM, otro IdP que redistribuye). Un mismo sistema puede ser ambos: un IdP que recibe eventos de un gestor de dispositivos y transmite sus propias consecuencias a aplicaciones downstream.

La relación se establece mediante un Stream, configurado por el Receiver contra el Transmitter vía una Stream Configuration Endpoint. En esa configuración el Receiver declara qué tipos de eventos quiere recibir, para qué subjects (usuarios, sesiones, dispositivos, tenants), y por qué canal de entrega: push HTTP POST hacia un endpoint del Receiver, o poll donde el Receiver pregunta periódicamente por eventos encolados.

Los subjects se identifican con Subject Identifiers del formato definido en RFC 9493, que permite referirse a un usuario por `iss`+`sub`, por email, por `phone_number`, por un identificador opaco del Transmitter, o por combinaciones tipo `aliases`. Esta flexibilidad es necesaria porque no siempre el Transmitter y el Receiver comparten el mismo identificador primario.

## El formato: Security Event Tokens

Los eventos viajan como SET (Security Event Tokens), definidos en RFC 8417. Un SET es un JWT con un claim `events` que contiene un objeto cuyas claves son URIs de tipo de evento y cuyos valores son los datos específicos de cada tipo. El SET se firma con la clave del Transmitter, y opcionalmente se cifra con JWE si el canal lo requiere.

El motivo de reutilizar JWT es práctico. Las implementaciones ya saben validar firmas JWT, rotar JWKs vía endpoint `.well-known`, manejar `iss`, `aud`, `iat`, `jti`. Añadir SET es un incremento pequeño sobre una base existente, no un protocolo nuevo desde cero. El parentesco con [JSON Web Token](/posts/JWT/) es evidente.

## Los eventos de CAEP

El perfil CAEP define un conjunto de eventos que cubren el ciclo de vida de una sesión desde la perspectiva del IdP. `session-revoked` notifica que una sesión ha sido invalidada (logout, expiración forzada, acción administrativa). `token-claims-change` informa de cambios en los claims que una aplicación debería reflejar (cambio de rol, cambio de grupos, cambio de atributos). `credential-change` avisa de que el usuario ha rotado una credencial (contraseña, clave FIDO, método MFA). `assurance-level-change` indica que el nivel de garantía de autenticación ha cambiado, típicamente porque el usuario subió o bajó un escalón de MFA. `device-compliance-change` refleja que la postura del dispositivo (gestionado, cifrado, con EDR activo) ha variado.

Cada evento incluye el subject afectado, un timestamp de cuándo ocurrió, y metadatos específicos. El receiver decide qué hacer con él: invalidar su caché de sesión, forzar re-autenticación, elevar el nivel de logging, bloquear acceso a recursos sensibles, o simplemente registrarlo para correlación en SIEM.

> CAEP no dicta la respuesta, solo transporta la señal. La política de qué hacer con `device-compliance-change: non-compliant` la decide el receiver según su modelo de riesgo.
{: .prompt-info }

## Entrega: push y poll

Push usa HTTP POST contra un endpoint que el Receiver expone durante la configuración del stream. Es baja latencia pero exige que el Receiver sea accesible por red desde el Transmitter, lo cual complica despliegues en redes privadas. Poll invierte la dirección: el Transmitter encola eventos y el Receiver los recupera llamando al Transmitter, lo cual encaja mejor en arquitecturas donde el consumidor vive detrás de firewalls.

Ambos modelos tienen que resolver problemas de entrega. El Transmitter retiene eventos hasta acuse explícito, reintenta ante errores, y permite al Receiver consultar el estado del stream. La duplicación de eventos es posible, así que los Receivers deben ser idempotentes: usar el `jti` del SET como clave para descartar repeticiones. La pérdida ordenada está acotada pero no imposible, así que eventos críticos suelen combinarse con verificación en el siguiente uso del token.

## Casos reales de despliegue

Google Cross-Account Protection fue el precursor y sigue siendo uno de los despliegues más visibles. Aplicaciones que usan "Sign in with Google" pueden suscribirse a eventos de compromiso de la cuenta Google del usuario (RISC) y actuar consecuentemente. Microsoft Continuous Access Evaluation está integrado en Entra ID y recursos como Exchange Online, SharePoint y Teams, propagando revocaciones y cambios de condición de acceso con latencias de segundos. Okta expone SSF tanto como Transmitter (notifica a apps integradas) como Receiver (puede consumir eventos de CrowdStrike, Jamf u otros para alimentar políticas de acceso).

## Relación con Zero Trust y con la gestión de sesiones

La promesa de [Zero Trust](/posts/Zero-Trust/) incluye una componente "continuous": verificar no solo al entrar sino durante toda la vida de la sesión. Sin un canal de eventos estandarizado, esa continuidad queda en polling costoso o en integraciones punto a punto. CAEP es la pieza que operativiza la idea. De manera análoga, [Gestión de sesiones y revocación](/posts/Sesiones-y-Revocacion/) describe mecanismos de revocación local; CAEP los conecta entre sistemas para que la revocación en el IdP se refleje en todos los SP.

El encaje con [OpenID Connect](/posts/OIDC/) es limpio: CAEP no sustituye OIDC, lo complementa. OIDC emite el token y establece la sesión, CAEP transporta los cambios posteriores. Lo mismo aplica a [SAML](/posts/SAML/), aunque en la práctica la adopción de CAEP está siendo empujada por ecosistemas OIDC-nativos.

## Cuándo vale la pena

CAEP tiene coste operativo real. Configurar streams, gestionar JWKs de transmitters, instrumentar receivers idempotentes, monitorizar entrega y retrasos, decidir políticas por tipo de evento. No es gratis. Vale la pena cuando el coste de una ventana de minutos entre que algo cambia en el IdP y se refleja en la aplicación es inaceptable: aplicaciones financieras, sistemas con acceso a datos personales sensibles, entornos regulados, plataformas con muchos IdPs externos cuyas sesiones tienen que ser controlables.

También vale la pena en ecosistemas grandes donde el polling no escala. Una SaaS con cientos de miles de tenants y múltiples IdPs externos no puede permitirse preguntar "sigue válido este token" cada minuto por cada sesión activa. El push con filtrado por subject es el único modelo que mantiene el coste acotado.

> Empezar suscribiéndose solo a `session-revoked` y `credential-change` cubre el 80 por ciento del valor con el 20 por ciento de la complejidad. Los eventos de assurance y device-compliance se añaden cuando el resto está sólido.
{: .prompt-tip }

## Cuándo no

Una aplicación pequeña, con un solo IdP, tokens cortos (5-10 minutos) y usuarios manejables a mano no necesita CAEP. El ciclo natural de refresco del token ya da un límite superior a la ventana de exposición, y ajustar esa caducidad es más barato que desplegar un framework de eventos. Tampoco encaja cuando las aplicaciones consumidoras son incapaces de exponer un endpoint fiable o de hacer polling periódico: en ese caso, mejor invertir en reducir caducidades que en construir una infraestructura de eventos a medias.

## Encaje en el ecosistema

CAEP se sitúa en la intersección entre el plano de autenticación (IdPs y federación), el plano de autorización (PDPs que reaccionan a cambios) y el plano de detección (SIEM, ITDR). Es la capa de transporte común para señales de identidad en tiempo real. A medida que más fabricantes lo implementan, el patrón "suscribirse a eventos del IdP" se convierte en la forma canónica de cerrar el bucle de Zero Trust sin inventar protocolos propietarios.

## Artículos relacionados en esta serie

- [Zero Trust](/posts/Zero-Trust/)
- [OpenID Connect](/posts/OIDC/)
- [Gestión de sesiones y revocación](/posts/Sesiones-y-Revocacion/)
- [ITDR](/posts/ITDR/)
- [Keycloak, Okta y los IdP modernos](/posts/Keycloak-Okta-IdP/)

## Referencias

- RFC 8417 (julio 2018). *Security Event Token (SET)*.
- RFC 9493 (diciembre 2023). *Subject Identifiers for Security Event Tokens*.
- OpenID Foundation (2024). *Shared Signals Framework Specification 1.0*.
- OpenID Foundation (2024). *Continuous Access Evaluation Profile 1.0*.
- OpenID Foundation (2024). *Risk and Incident Sharing and Coordination Profile 1.0*.
- Microsoft (2022). *Continuous Access Evaluation in Microsoft Entra ID*.
