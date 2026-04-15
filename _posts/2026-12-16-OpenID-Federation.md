---
title: "OpenID Federation: federación de IdPs a escala"
author: Xabierland
description: >-
    OpenID Federation define cómo múltiples proveedores de identidad, aplicaciones y autoridades intermedias establecen confianza mutua sin intercambio manual de metadatos. Se basa en cadenas de entity statements firmados que permiten resolución dinámica hasta un trust anchor común. Es la base técnica de ecosistemas como la European Digital Identity Wallet de eIDAS 2.0.
date: 2026-12-16 10:00
categories: [DevSecOps, Seguridad de Aplicaciones]
tags: [OpenID, Federacion, Confianza, eIDAS, JWT, IdP, RFC9678]
---

Este artículo forma parte de la serie [La evolución de la seguridad en aplicaciones modernas](/posts/La-evolucion-de-la-seguridad-en-aplicaciones-modernas/).

Federar dos sistemas de identidad es un problema resuelto. Federar diez ya empieza a ser tedioso. Federar cien, donde cualquiera del conjunto puede necesitar confiar dinámicamente en cualquier otro, y donde entran y salen participantes con regularidad, rompe los modelos clásicos basados en intercambio manual de metadatos. OpenID Federation, formalizado en RFC 9678 en octubre de 2024, ofrece una respuesta basada en cadenas de declaraciones firmadas que se resuelven en tiempo real hasta una raíz de confianza común.

## El problema que SAML y OIDC clásico dejan abierto

En SAML, federar dos entidades implica intercambiar un Metadata XML firmado que describe endpoints, certificados y políticas. Funciona bien entre dos partes, pero no escala: cada participante mantiene una lista de metadatos de todos los demás, y añadir o rotar uno obliga a actualizar a todo el mundo. Los proyectos académicos como eduGAIN resolvieron este problema a lo bruto con una federación de federaciones que agrega metadatos de miles de entidades en un único documento distribuido diariamente, con firmado centralizado.

En OpenID Connect estándar, el Discovery Document publicado en `/.well-known/openid-configuration` y el JWKS asociado cumplen una función similar para relaciones bilaterales: el cliente OIDC descubre endpoints y claves del IdP. Pero no hay una forma estandarizada de decir "confío en este IdP porque una autoridad superior me lo dice", ni de establecer confianza transitiva entre participantes que no tienen relación previa.

OpenID Federation cubre ese hueco con un modelo pensado desde el principio para ecosistemas abiertos.

## Entity statements como bloque básico

La pieza fundamental es el entity statement, un JWT firmado que una entidad emite sobre sí misma o sobre otra entidad. Cada participante en una federación publica un entity configuration (statement sobre sí mismo) en un endpoint `.well-known/openid-federation`. Ese statement incluye las claves públicas de la entidad, sus metadatos OIDC (endpoints, client info), y, crucialmente, una lista de authority_hints: los identificadores de autoridades superiores que pueden tener statements sobre ella.

Las autoridades intermedias, a su vez, publican entity statements sobre las entidades que les son subordinadas. Una universidad que participa en una federación académica publica su configuración y lista la federación como authority_hint. La federación publica un statement sobre la universidad que le declara miembro y la autoriza a ciertos tipos de interacción. Si la federación está a su vez bajo una federación de federaciones, la cadena sigue hacia arriba hasta llegar a un trust anchor: una entidad cuya clave pública está configurada out-of-band en todos los participantes como raíz ultima de confianza.

## Trust chains: resolución dinámica

Cuando un relying party recibe una solicitud o quiere validar un IdP con el que no tenía relación previa, no consulta una lista estática. En su lugar, construye una trust chain: parte del entity configuration del IdP, sigue sus authority_hints, obtiene entity statements que acreditan al IdP, sigue las authority_hints de esas autoridades, y así sucesivamente hasta encontrar un trust anchor que el RP ya conoce. Si la cadena se cierra y todas las firmas son válidas, la confianza queda establecida.

El mecanismo permite que dos participantes sin acuerdo bilateral puedan verificarse con base en que comparten una raíz común. La cadena resuelta produce, como resultado, un metadata policy merged: los metadatos del IdP filtrados y restringidos por las políticas que cada autoridad superior impone. Una federación puede limitar, por ejemplo, qué response_types o qué scopes están autorizados para sus miembros, y esa restricción se propaga hasta el metadata efectivo que el RP termina usando.

> El trust anchor es la raíz física del modelo: su clave pública tiene que estar distribuida por canales fuera del protocolo. Si alguien controla el trust anchor, controla toda la federación.
{: .prompt-warning }

## Registro de clientes: explícito vs automático

El modelo clásico de OIDC obliga a cada cliente a registrarse contra cada IdP antes de usarlo. En ecosistemas grandes, ese registro manual es el cuello de botella. OpenID Federation define dos modos de registro. El explícito sigue siendo posible: un cliente se registra formalmente contra un IdP federado, el IdP valida la trust chain del cliente, y emite credenciales. El automático es más radical: no hay registro previo. Cuando el cliente aparece por primera vez en una authorization request, presenta su entity configuration (o un puntero a ella), el IdP resuelve la trust chain, y si se cierra contra un trust anchor común, el cliente queda implícitamente registrado para esa sesión o de forma persistente según política.

Automatic Client Registration es, con diferencia, la propiedad más poderosa para ecosistemas abiertos. Un nuevo relying party que se incorpora a la federación puede empezar a trabajar contra cualquier IdP federado sin que cada IdP tenga que hacer nada explícito.

## Trust marks

Las federaciones a menudo necesitan expresar que ciertos miembros cumplen requisitos adicionales: auditoría de seguridad superada, cumplimiento de un nivel de assurance, certificación sectorial específica. OpenID Federation modela esto con trust marks: claims firmados que una autoridad designada puede emitir sobre una entidad y que esta publica en su entity configuration. El RP puede condicionar su confianza no solo a que la cadena cierre, sino a que existan trust marks concretos emitidos por autoridades aceptadas.

## Casos reales

El despliegue más ambicioso en curso es el ecosistema de la European Digital Identity Wallet bajo eIDAS 2.0. El Architecture and Reference Framework de la Comisión Europea identifica OpenID Federation como uno de los mecanismos principales para establecer confianza entre los componentes del ecosistema: wallets, issuers de atributos, relying parties, autoridades de confianza de cada estado miembro y el trust framework europeo. La escala prevista (cientos de millones de ciudadanos, miles de emisores y verificadores distribuidos en 27 estados miembros) hace inviable cualquier modelo estático.

GAIA-X, la iniciativa europea de espacios de datos federados, explora patrones similares para identidad de servicios y organizaciones. Las federaciones académicas, históricamente ancladas en SAML via eduGAIN, están en proceso de evaluación o migración hacia OpenID Federation para cubrir los casos de uso OIDC modernos sin perder el modelo de confianza multinivel que ya funciona.

## Relación con OIDC estándar

OpenID Federation no sustituye a [OpenID Connect](/posts/OIDC/). Es una capa encima que define cómo se establece la confianza entre las entidades OIDC; el flujo de autenticación, los tokens, los scopes, siguen siendo los mismos. Un despliegue puede hacer OIDC puro entre un par de actores y OpenID Federation entre muchos, o incluso combinar ambos en el mismo IdP según el cliente.

La relación con [JSON Web Token](/posts/JWT/) es también directa: entity statements son JWTs firmados, con las mismas reglas de validación, rotación de claves (publicadas en JWKS dentro del propio statement) y expiración. Cualquier librería capaz de validar JWTs bien puede procesar entity statements con esfuerzo modesto.

## Complejidad operativa

El modelo es potente pero no barato. Cada participante tiene que publicar y mantener su entity configuration, rotar sus claves con política clara, actualizar authority_hints cuando la estructura de la federación cambia. Las autoridades intermedias emiten statements sobre sus subordinados y los renuevan antes de caducar; un statement expirado rompe la trust chain y deja a sus subordinados invisibles. Las relying parties cachean trust chains con políticas de TTL que balancean frescura y carga, y tienen que manejar rollover de claves en autoridades superiores sin interrupciones.

El tooling maduro es todavía limitado. SpID en Italia, algunos pilotos nórdicos y los SDK de referencia de la OpenID Foundation están entre los ejemplos más avanzados. Fuera de proyectos alineados con eIDAS 2.0 o federaciones académicas, los despliegues de producción son aún contados.

> OpenID Federation brilla cuando hay autoridad común reconocida por todos. Sin un trust anchor natural y aceptado, el modelo pierde su razón de ser.
{: .prompt-info }

## Cuándo lo necesitas

Tres condiciones hacen OpenID Federation la respuesta adecuada. Primera, un número de participantes suficientemente grande o creciente como para que el intercambio bilateral no escale. Segunda, existencia de una autoridad o jerarquía de autoridades que pueda actuar como trust anchor con legitimidad reconocida por todos: un gobierno, un regulador sectorial, una asociación profesional, un consorcio. Tercera, necesidad de que participantes nuevos se incorporen sin rondas bilaterales de firma de acuerdos y configuración manual.

Ecosistemas como identidad digital nacional (eIDAS 2.0 siendo el caso paradigmático), federaciones educativas, ecosistemas sanitarios entre múltiples proveedores, espacios de datos sectoriales, encajan en este patrón.

## Cuándo no

Una federación bilateral simple entre dos organizaciones no gana nada con OpenID Federation. El intercambio manual del Discovery Document y el JWKS ya resuelve el problema con una fracción de la complejidad. Un IdP central con muchos clientes internos tampoco: el registro estático contra un único IdP es más directo que montar entity statements y authority_hints para un modelo de una sola autoridad. Igualmente, si el ecosistema carece de un trust anchor natural aceptado, inventarlo artificialmente crea más problemas de gobernanza de los que resuelve técnicamente.

## Encaje en el ecosistema

OpenID Federation es la capa que permite a los protocolos OIDC existentes escalar a ecosistemas de confianza abiertos. Conecta con [OpenID Connect](/posts/OIDC/) como mecanismo base de autenticación, con [JSON Web Token](/posts/JWT/) como formato, y se complementa con [Verifiable Credentials, DIDs y SSI](/posts/Verifiable-Credentials-DIDs-SSI/) donde las credenciales presentadas por el usuario pueden venir de emisores descubiertos vía la propia federación. En el horizonte regulatorio europeo, es una de las piezas técnicas que materializa la promesa de identidad digital interoperable transfronteriza.

## Artículos relacionados en esta serie

- [OpenID Connect](/posts/OIDC/)
- [JSON Web Token](/posts/JWT/)
- [Keycloak, Okta y los IdP modernos](/posts/Keycloak-Okta-IdP/)
- [SAML](/posts/SAML/)
- [Verifiable Credentials, DIDs y SSI](/posts/Verifiable-Credentials-DIDs-SSI/)

## Referencias

- RFC 9678 (octubre 2024). *OpenID Federation 1.0*.
- OpenID Foundation (2024). *OpenID Federation Wallet Architectures*.
- Comisión Europea (2024). *European Digital Identity Architecture and Reference Framework 1.4*.
- Parlamento Europeo (abril 2024). *Reglamento (UE) 2024/1183 por el que se modifica el Reglamento (UE) n.º 910/2014 (eIDAS 2.0)*.
- RFC 7519 (mayo 2015). *JSON Web Token (JWT)*.
