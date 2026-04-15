---
title: "FIDO2, WebAuthn y Passkeys: el fin de la contraseña"
author: Xabierland
description: >-
    FIDO2 es el conjunto de especificaciones de FIDO Alliance y W3C que define autenticación basada en criptografía asimétrica ligada al origen, sin contraseñas compartidas. Resuelve el problema estructural del phishing y del credential stuffing, imposibles por construcción cuando no existe un secreto transmitido. Passkeys es la marca con la que Apple, Google y Microsoft llevaron esa tecnología al gran público desde 2022, convirtiendo lo que era propiedad de entornos de alta seguridad en el mecanismo de login por defecto de la web.
date: 2026-10-21 10:00
categories: [DevSecOps, Seguridad de Aplicaciones]
tags: [FIDO2, WebAuthn, Passkeys, Autenticación, Criptografía, CTAP2, Phishing]
---

Este artículo forma parte de la serie [La evolución de la seguridad en aplicaciones modernas](/posts/La-evolucion-de-la-seguridad-en-aplicaciones-modernas/). La idea de matar la contraseña es vieja, y hasta hace poco se quedaba en buenas intenciones. Lo que cambió con FIDO2 y WebAuthn es que, por primera vez, la infraestructura (navegadores, sistemas operativos, silicio seguro en los dispositivos) alineó todas las piezas. Passkeys es la narrativa final: no pedir al usuario que entienda criptografía, sino que haga exactamente lo que ya sabe hacer (desbloquear el teléfono) y que eso baste para entrar en cualquier sitio. En este artículo se explica de dónde viene, cómo funciona la pieza criptográfica, cuáles son los compromisos entre passkeys sincronizadas y device-bound, y cuándo conviene usar cada modelo.

## Por qué existe FIDO2

FIDO Alliance se formó en 2013 con un objetivo explícito: eliminar el secreto compartido de los flujos de autenticación. La primera generación de estándares (U2F como segundo factor, UAF para autenticación sin contraseña) demostró la viabilidad técnica en nichos concretos, sobre todo con security keys USB tipo YubiKey. La segunda generación, FIDO2, anunciada en 2018 y completada con la recomendación W3C de WebAuthn de marzo de 2019, llevó la misma idea a dos piezas estandarizadas y universalmente disponibles: WebAuthn como API del navegador, y CTAP2 (Client to Authenticator Protocol v2) como protocolo entre el navegador y el autenticador (sea plataforma, sea roaming por USB/NFC/BLE).

El problema que resuelve es doble. Primero, el phishing: si no hay secreto que el usuario pueda entregar, no hay phishing posible en el sentido tradicional. Segundo, las brechas masivas: si el servidor no almacena un verificador de contraseña sino una clave pública, un volcado de la base de datos no concede nada utilizable contra otros servicios ni siquiera contra este. La combinación es lo que hace a FIDO2 cualitativamente distinto de MFA tradicional.

## La pieza criptográfica

El modelo es criptografía asimétrica por relying party. Cuando un usuario se registra en un sitio (el relying party, RP), el navegador pide al autenticador generar un par de claves nuevo, exclusivo para ese RP. El autenticador guarda la clave privada y devuelve la clave pública al RP, junto con un identificador de credencial. El RP almacena clave pública y credential ID asociados al usuario. No hay secreto compartido. No hay contraseña. La clave privada no sale nunca del autenticador.

En cada login, el RP envía un challenge aleatorio. El autenticador lo firma con la clave privada correspondiente a ese RP, previa verificación del usuario (huella, PIN, rostro). El navegador entrega la firma al RP, que la valida con la clave pública almacenada. Dos detalles hacen esta construcción resistente a phishing por diseño. El navegador añade al challenge el origen real de la página que lo solicita (client data) y lo mete dentro del material firmado. Y el autenticador asocia cada credencial al RP ID, de modo que incluso si un sitio falso consigue convencer al usuario, el autenticador no producirá una firma válida para el RP original, simplemente porque la credencial que ese sitio pide no existe en él.

CTAP2 cubre cómo el navegador habla con el autenticador externo (USB, NFC, BLE). En autenticadores de plataforma (Touch ID, Windows Hello, Android Keystore) el diálogo es interno al sistema operativo. El resultado, desde el punto de vista del servidor, es el mismo: una firma sobre un challenge con contexto de origen.

## Atestación y tipos de autenticador

Durante el registro, el autenticador puede enviar una attestation statement, firmada con una clave raíz del fabricante, que permite al RP verificar qué modelo de autenticador se usó y sus propiedades (por ejemplo, si la clave vive en hardware seguro, si soporta user verification, si es roaming o plataforma). Los formatos estándar son packed, tpm, android-key, android-safetynet, fido-u2f y none. El modo none se usa cuando el RP no necesita saber el tipo de autenticador y prefiere privacidad; muchos despliegues consumer lo eligen por defecto.

> La atestación es potente en entornos corporativos donde el RP quiere aceptar solo autenticadores certificados de determinados fabricantes (por ejemplo, FIDO2 L2 o L3). En entornos consumer suele evitarse porque correlaciona usuarios entre sitios y complica la adopción.
{: .prompt-info }

Los autenticadores se clasifican en platform (integrados en el dispositivo: Secure Enclave, TPM, Titan M) y roaming (externos: security keys). Los platform dan la mejor UX porque no exigen un objeto adicional; los roaming dan portabilidad entre dispositivos y, en sus variantes hardware-bound, el máximo nivel de resistencia.

## User verification y user presence

WebAuthn distingue user presence (UP: el usuario hizo algún gesto físico, típicamente tocar la key) de user verification (UV: el autenticador verificó activamente la identidad del usuario mediante biometría o PIN). Un flujo completo FIDO2 passwordless exige UV. Un flujo FIDO2 como segundo factor puede aceptar UP solo. Esta distinción permite a un RP expresar políticas: para login inicial, UV; para reautenticar una acción sensible, UV obligatorio; para confirmar una operación, quizá basta con UP.

## Passkeys: la evolución consumer

En mayo de 2022 Apple, Google y Microsoft anunciaron conjuntamente soporte alineado a Passkeys. La idea técnica es sencilla y clave para la adopción masiva: las credenciales FIDO2 que tradicionalmente vivían atadas a un autenticador concreto (device-bound) pueden ahora sincronizarse a través del llavero del proveedor (iCloud Keychain, Google Password Manager, Windows Hello con sincronización). El usuario que da de alta una passkey en su iPhone la encuentra disponible en su iPad y en su Mac. Si pierde el dispositivo, no pierde la credencial.

La sincronización se hace end-to-end cifrada por el proveedor del llavero, con flujos de recuperación basados en el mecanismo de cuenta de ese proveedor (Apple ID, cuenta Google, cuenta Microsoft). Desde el punto de vista del RP, la credencial sigue siendo una clave pública. Lo que cambia es el modelo de amenaza: la clave privada ya no está ligada a una pieza concreta de hardware, sino al control de la cuenta del proveedor del llavero.

El despliegue masivo vino entre 2023 y 2024. Google anunció passkeys como opción de login por defecto en cuentas personales. Apple las integró nativamente en Safari y en el llavero. Microsoft las llevó a cuentas personales y corporativas vía Entra ID. Sitios grandes (Amazon, PayPal, GitHub, WhatsApp, grandes bancos) las ofrecen como alternativa a contraseña, y algunos como sustituto. El Hybrid Transport de CTAP2 (también conocido como cross-device authentication, con escaneo de QR y BLE proximity) permite usar la passkey del teléfono para iniciar sesión en un navegador de otro dispositivo.

## Sincronizadas vs device-bound: el trade-off

No todas las passkeys son sincronizadas. FIDO Alliance mantiene la distinción explícita entre syncable passkeys (las del llavero) y device-bound passkeys (las que viven en una security key hardware o en un TPM/Secure Enclave sin exportar). Las syncable resuelven el problema de pérdida de dispositivo al precio de depender del proveedor del llavero y de su modelo de recuperación de cuenta. Las device-bound mantienen la propiedad de que la clave privada no sale nunca de ese hardware concreto, al precio de exigir procedimientos de enrollment de segundo dispositivo y de backup key.

Para banca en línea de consumo, correo personal, redes sociales, las passkeys sincronizadas son la elección casi siempre correcta: la usabilidad es la que hace la adopción. Para administradores de dominio, para root de cuentas AWS, para acceso a HSMs, para bastion hosts, las device-bound siguen siendo imprescindibles. FIDO2 Security Key Level 3 y las guías corporativas de passwordless suelen exigir device-bound precisamente para romper la dependencia del ecosistema del llavero.

## Cuándo tiene sentido y cuándo no

Tiene sentido adoptar FIDO2/passkeys como factor primario siempre que la tasa esperada de pérdida de acceso legítimo sea gestionable y exista un procedimiento de recuperación razonable. En consumer con passkeys sincronizadas la recuperación la da el proveedor del llavero. En corporativo con device-bound, el procedimiento es enrollment de una segunda key y escrow de recuperación operado por el help desk.

No tiene sentido imponerlas como único factor en poblaciones donde el ecosistema no llega: usuarios sin smartphone moderno, quioscos compartidos, entornos industriales con políticas restrictivas de dispositivos personales. En esos casos conviene ofrecerlas como opción y mantener rutas alternativas (TOTP, security keys roaming compartidas bajo custodia) con la fricción que corresponda.

> El error más repetido al desplegar passkeys es olvidar el path de offboarding: qué pasa cuando el usuario deja la organización, cuando cambia de cuenta Apple, cuando pierde el dispositivo y el llavero simultáneamente. Diseña esos flujos antes de forzar passkeys como único factor.
{: .prompt-warning }

## Encaje con el resto de la arquitectura

WebAuthn es la pieza de autenticación; la autorización sigue viviendo en [OAuth 2.0](/posts/OAuth2/) y [OIDC](/posts/OIDC/). Un IdP moderno expone WebAuthn/passkeys como método de autenticación primaria, emite los tokens correspondientes, y comunica a las aplicaciones el nivel de garantía vía acr y amr como se describe en [MFA](/posts/MFA/). En [Zero Trust](/posts/Zero-Trust/), passkeys es el factor que hace viable asumir autenticación continua fuerte sin penalizar al usuario en cada acceso. Frente a flujos legados con [Kerberos](/posts/Kerberos/) y directorio LDAP, es el escalón moderno que cierra la historia de la contraseña con algo mejor que una recomendación, una especificación realmente desplegable.

## Artículos relacionados en esta serie

- [MFA](/posts/MFA/)
- [OpenID Connect](/posts/OIDC/)
- [Keycloak, Okta y los IdP modernos](/posts/Keycloak-Okta-IdP/)
- [Zero Trust](/posts/Zero-Trust/)

## Referencias

- W3C (marzo 2019, revisiones posteriores). *Web Authentication: An API for accessing Public Key Credentials*.
- FIDO Alliance (2018, actualizaciones posteriores). *Client to Authenticator Protocol (CTAP) Specification v2*.
- FIDO Alliance (2022). *Multi-Device FIDO Credentials (Passkeys) White Paper*.
- NIST (2017, revisiones sucesivas). *SP 800-63B: Digital Identity Guidelines - Authentication and Lifecycle Management*.
- Apple, Google, Microsoft (mayo 2022). *Joint announcement on passwordless sign-in support across devices and platforms*.
