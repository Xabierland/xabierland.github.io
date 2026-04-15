---
title: "Verifiable Credentials, DIDs y SSI: el horizonte de la identidad descentralizada"
author: Xabierland
description: >-
    SSI (Self-Sovereign Identity) propone un modelo donde el sujeto controla sus credenciales sin depender de un IdP central. Resuelve la acumulación de datos en proveedores y la dependencia de un único emisor para cada identidad. En el ecosistema moderno se materializa con los estándares W3C de DIDs y Verifiable Credentials, OpenID4VC y regulaciones como eIDAS 2.0.
date: 2026-11-18 10:00
categories: [DevSecOps, Seguridad de Aplicaciones]
tags: [SSI, DID, Verifiable Credentials, W3C, eIDAS, OpenID4VC, Identidad descentralizada]
---

Este artículo forma parte de la serie [La evolución de la seguridad en aplicaciones modernas](/posts/La-evolucion-de-la-seguridad-en-aplicaciones-modernas/).

Durante dos décadas, la identidad digital se construyó alrededor de proveedores centrales: primero corporativos, después las grandes plataformas sociales y, en paralelo, los IdP empresariales. El modelo funciona, pero concentra poder y datos en un puñado de actores. La Self-Sovereign Identity propone dar la vuelta al guion: que el sujeto guarde sus credenciales en su propio dispositivo y decida qué presenta, a quién y con qué granularidad.

## El problema que intenta resolver

En el modelo actual, cada servicio pide su propia prueba de identidad. Una universidad emite un título y vive para siempre en un PDF que nadie puede verificar con confianza. Un banco pide el mismo DNI una y otra vez para operaciones básicas. Un estado emite un carnet físico y, para demostrarlo en línea, hay que subir una foto a un tercero que hará OCR con calidad variable. Cada intercambio produce datos que terminan en bases de datos ajenas al titular y expuestas a brechas.

La idea de SSI es que el titular reciba la credencial una vez, la guarde en un wallet bajo su control y la presente bajo demanda con una firma criptográfica verificable por cualquiera que conozca el emisor. El verificador no necesita llamar al emisor en cada consulta: basta con comprobar la firma y el estado de revocación. El emisor no sabe cuándo ni dónde se usa la credencial, porque no interviene en la verificación. El titular decide qué atributos revelar: edad mayor a dieciocho sin exponer la fecha de nacimiento, por ejemplo.

## Las tres piezas del modelo W3C

El marco se construye sobre tres estándares. Los DIDs (Decentralized Identifiers) son identificadores autosoberanos publicados en un registro verificable. La especificación es W3C Recommendation desde junio de 2022. Un DID no apunta a una cuenta en un proveedor; apunta a un documento que contiene las claves públicas del sujeto y los endpoints de servicio. Existen múltiples métodos de resolución: did:web (publicado en un dominio HTTPS), did:key (el DID es la clave pública), did:ion (sobre Bitcoin mediante Sidetree), did:ebsi (sobre la blockchain europea EBSI) y decenas más.

Verifiable Credentials Data Model v2.0, W3C Recommendation desde mayo de 2025, define la estructura de las credenciales: quién es el emisor (otro DID), quién es el titular, qué afirma, cuándo caduca y la firma criptográfica. El modelo es agnóstico del formato: las credenciales pueden serializarse como JSON-LD con Linked Data Proofs, como JWT-VC o como SD-JWT.

La tercera pieza, las Verifiable Presentations, empaqueta una o varias credenciales seleccionadas por el titular junto con una prueba de posesión, firmada con la clave del propio titular. Es el artefacto que se envía al verificador en cada interacción.

## Formatos y selective disclosure

No todas las credenciales se representan igual. JWT-VC es la opción pragmática para equipos que vienen del mundo [JSON Web Token](/posts/JWT/): una credencial es un JWT firmado con los claims del modelo VC. SD-JWT (Selective Disclosure JWT), especificado en drafts del IETF, permite al emisor firmar una credencial con atributos individualmente ocultables, de modo que el titular revele solo los campos necesarios y el verificador pueda comprobar la firma sin ver el resto.

SD-JWT resuelve un problema práctico importante: sin selective disclosure, presentar una credencial obliga a revelar todos los atributos, lo que choca con principios de minimización de datos. Para casos como la verificación de edad en plataformas en línea, poder demostrar "mayor de edad" sin exponer fecha exacta, dirección ni número de documento es la diferencia entre cumplir RGPD y no cumplirlo.

Las pruebas de conocimiento cero más sofisticadas (BBS+, Groth16) permiten incluso revelar predicados derivados sin que el emisor tenga que firmarlos individualmente. Su adopción va por detrás de SD-JWT por coste computacional y madurez de librerías, pero están en el horizonte.

## OpenID4VC: el puente con lo que ya existe

Para que el ecosistema SSI se integre con la infraestructura actual de identidad hacía falta un puente con [OAuth 2.0](/posts/OAuth2/) y [OpenID Connect](/posts/OIDC/). La respuesta es la familia OpenID for Verifiable Credentials, desarrollada en la OpenID Foundation. OpenID4VCI describe cómo un titular pide una credencial a un emisor usando flujos OAuth, y OpenID4VP define cómo un verificador solicita una presentación al wallet del titular, con Presentation Exchange como lenguaje para expresar qué atributos se requieren.

La elección no es casual: los equipos que ya implementan OIDC pueden añadir soporte VC reutilizando autenticación, consentimiento y capa de transporte. Esto baja la barrera de entrada y convierte a los IdP tradicionales en emisores y verificadores de credenciales, no en piezas reemplazadas.

> A corto plazo, SSI no sustituye al IdP corporativo. Lo extiende. Los proyectos que arrancan con la promesa de "eliminar Keycloak" suelen volver a reintroducirlo para tareas que las VC no cubren bien, como gestión de sesiones interactivas.
{: .prompt-warning }

## eIDAS 2.0 y la EUDI Wallet

El mayor impulso regulatorio llega de la Unión Europea. El Reglamento UE 2024/1183, conocido como eIDAS 2.0, obliga a cada estado miembro a ofrecer una European Digital Identity Wallet (EUDI Wallet) a sus ciudadanos, con un calendario que sitúa las primeras emisiones masivas a partir de finales de 2026. La wallet deberá almacenar la identidad nacional, permitir almacenar credenciales de terceros y funcionar como autenticador frente a servicios públicos y privados.

La implementación técnica se apoya en los Architecture Reference Framework publicados por la Comisión, que consolidan OpenID4VCI, OpenID4VP y SD-JWT como los formatos obligatorios en la primera fase, con ISO/IEC 18013-5 (mobile Driving Licence, mDL) para credenciales que requieran presentación offline por proximidad NFC o Bluetooth.

Para el sector privado esto significa dos cosas: los servicios que hoy piden DNI electrónico o sistemas nacionales de identidad deberán aceptar la EUDI Wallet cuando operen en la UE, y aparece una oportunidad clara para emitir credenciales sectoriales (títulos, licencias profesionales, afiliaciones) reutilizando el mismo stack.

## Casos de uso que ya funcionan

Los títulos académicos son el caso paradigmático: una universidad emite una credencial VC, el graduado la guarda en su wallet y la presenta a un empleador que la verifica sin llamar al registro nacional. El programa Europass Digital Credentials ya emite credenciales bajo este modelo, y varias universidades europeas lo han adoptado.

El carnet de conducir móvil, bajo ISO/IEC 18013-5 mDL, es el segundo caso maduro. Estados de Estados Unidos como Utah, Arizona y California emiten mDL aceptadas en puntos de control TSA. En Europa, varios pilotos de EUDI Wallet ya integran la licencia de conducir como credencial nativa.

El tercer caso emergente es el KYC reutilizable. Una entidad financiera realiza la verificación completa de un cliente y emite una VC que acredita "KYC verificado a tal fecha con tal nivel". Otras entidades aceptan esa credencial sin repetir el proceso, reduciendo fricción y duplicación. El reto no es técnico sino de gobernanza: qué frameworks aceptan qué emisores.

## Trade-offs y riesgos

La complejidad criptográfica es real. Implementar un wallet que gestione claves, varios métodos DID, varios formatos de credencial y protocolos de presentación no es un proyecto trivial. La UX es un problema abierto: mover al usuario desde "usuario y contraseña" a "abra su wallet y escanee el QR" requiere aprendizaje. Los ataques de phishing mutan: en vez de robar credenciales, inducen al titular a presentar credenciales reales a verificadores maliciosos.

> La gobernanza de los trust frameworks es hoy el cuello de botella. Un verificador necesita saber qué emisores acepta para cada tipo de credencial, y esa lista evoluciona. Sin un catálogo común, el riesgo de fragmentación por país y por sector es alto.
{: .prompt-info }

## Encaje en el stack

SSI no reemplaza [Keycloak, Okta y los IdP modernos](/posts/Keycloak-Okta-IdP/), ni vuelve obsoleto a [OAuth 2.0](/posts/OAuth2/). Se integra como una capa de credenciales portables sobre los protocolos existentes, gracias a OpenID4VC. El IdP corporativo puede convertirse en emisor de VC para empleados y clientes, y simultáneamente en verificador de credenciales emitidas por terceros. La transición es evolutiva, no disruptiva, y los primeros cinco años serán de convivencia más que de sustitución.

## Artículos relacionados en esta serie

- [OpenID Connect](/posts/OIDC/)
- [OAuth 2.0](/posts/OAuth2/)
- [Keycloak, Okta y los IdP modernos](/posts/Keycloak-Okta-IdP/)
- [FIDO2, WebAuthn y Passkeys](/posts/FIDO2-WebAuthn-Passkeys/)
- [JSON Web Token](/posts/JWT/)

## Referencias

- W3C (junio 2022). *Decentralized Identifiers (DIDs) v1.0*.
- W3C (mayo 2025). *Verifiable Credentials Data Model v2.0*.
- Unión Europea (2024). *Reglamento (UE) 2024/1183 por el que se modifica el Reglamento eIDAS*.
- OpenID Foundation (2024). *OpenID for Verifiable Credential Issuance y OpenID for Verifiable Presentations*.
- ISO/IEC 18013-5 (2021). *Personal identification — ISO-compliant driving licence — Part 5: Mobile driving licence (mDL)*.
