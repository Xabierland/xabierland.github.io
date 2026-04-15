---
title: "Entitlements as a Service: AuthZed, Cerbos, Permit y la autorización gestionada"
author: Xabierland
description: >-
    EaaS (Entitlements as a Service) es la oferta de autorización como servicio gestionado, con control plane, SDKs y UI. Resuelve el coste operativo de desplegar un motor de políticas propio y el gobierno fragmentado entre aplicaciones. En el ecosistema moderno compite y complementa a OPA, OpenFGA y motores embebidos, dibujando un mapa de trade-offs entre latencia, control y velocidad de entrega.
date: 2026-11-04 10:00
categories: [DevSecOps, Seguridad de Aplicaciones]
tags: [Autorización, AuthZed, Cerbos, Permit, Oso, OPA, OpenFGA, SaaS]
---

Este artículo forma parte de la serie [La evolución de la seguridad en aplicaciones modernas](/posts/La-evolucion-de-la-seguridad-en-aplicaciones-modernas/).

La autorización pasó en pocos años de ser un `if` en el controlador a un dominio propio con motores, lenguajes de políticas y modelos relacionales. El siguiente paso lógico era empaquetar ese dominio como producto gestionado. EaaS (Entitlements as a Service) es esa propuesta: un proveedor opera el control plane de políticas, el catálogo de relaciones y el PDP (Policy Decision Point), y la aplicación solo consume decisiones mediante un SDK.

## Por qué emergió la autorización gestionada

Cuando una organización adopta [CASBIN, OPA y OpenFGA](/posts/CASBIN-OPA-OpenFGA/) o cualquier motor embebido, descubre rápido que la pieza de cómputo no es la más costosa. El gasto real está en lo que rodea al motor: el pipeline de CI que prueba las políticas, el sistema de versionado, la distribución a decenas de servicios, el tablero para que producto o legal modifiquen reglas sin desplegar código, el SDK idiomático para cada lenguaje, la observabilidad de decisiones, la caché local para mantener latencias sub-milisegundo, el bundle de relaciones para modelos ReBAC. Multiplicar esto por cada aplicación del catálogo y por cada equipo hace que el coste se dispare.

La propuesta de EaaS es desacoplar ese esfuerzo del negocio: se paga una suscripción y se obtiene lo que antes había que construir. Es el mismo patrón que ya ocurrió con bases de datos gestionadas, message brokers gestionados o identity providers gestionados. La autorización llega tarde a esa ola porque es especialmente sensible a la latencia y a la residencia de datos, pero llega.

## El mapa de actores

El mercado se organiza en varios clusters con enfoques distintos.

AuthZed, creadora de SpiceDB, es la propuesta más cercana al paper de Google Zanzibar publicado en 2019. Ofrece un modelo puramente ReBAC donde todo es una relación entre objetos, con consistencia basada en zookies (tokens opacos que describen el punto de consistencia de una relación). Su versión gestionada, AuthZed Dedicated y Serverless, aparecieron a partir de 2021 sobre el motor de código abierto SpiceDB.

Cerbos, lanzado en 2021, escogió la vía contraria: políticas declarativas en YAML con expresiones condicionales, ejecutadas contra atributos del principal y del recurso que la aplicación envía en cada consulta. Es efectivamente ABAC estructurado, sin necesidad de mantener un grafo de relaciones. Su producto Cerbos Hub añade control plane, distribución de bundles firmados y playground.

Permit.io se posicionó como capa encima de motores existentes, construyendo una UI no-code sobre OPA y OpenFGA. El atractivo es que producto o compliance pueden modelar políticas visualmente y Permit traduce internamente a Rego o al DSL de OpenFGA. Es la opción con más énfasis en ergonomía del no técnico.

Oso, que empezó en 2019 con el lenguaje Polar y un motor embebido en la aplicación, pivotó en 2023 a un servicio gestionado centrado en listas de datos y relaciones, abandonando progresivamente la distribución embebida. Aserto, fundada por exempleados de Microsoft, construye sobre OPA con su motor Topaz y añade gestión de objetos. Warrant, adquirida por WorkOS en 2024, incorporó el patrón Zanzibar al stack de WorkOS.

## Arquitectura típica

Aunque los detalles varían, el patrón se repite. Hay un control plane en la nube donde se almacenan políticas, esquemas y, en los sistemas ReBAC, las relaciones. Las aplicaciones consumen decisiones a través de un PDP que se despliega como sidecar, como contenedor cercano o como servicio remoto. Ese PDP mantiene una caché local de políticas y, cuando la latencia lo exige, también de un subconjunto de relaciones. Las decisiones emiten logs estructurados que suben al control plane para auditoría y analítica.

La diferencia más relevante entre propuestas es dónde vive la decisión. Si el PDP corre en el mismo pod que la aplicación, la latencia puede bajar del milisegundo pero el control plane sigue viendo los logs; si el PDP es un servicio remoto del proveedor, la latencia sube a cinco o diez milisegundos pero la operación es más sencilla. Cerbos y OpenFGA favorecen PDP local; AuthZed combina ambos modos según el plan.

> No hay elección universal entre PDP local y remoto. La decisión depende del SLO de latencia de las APIs que autorizan y del volumen de políticas que cada pod debe cargar en memoria.
{: .prompt-info }

## Trade-offs reales

La latencia es la variable más visible. Un PDP embebido sirve decisiones en microsegundos; un servicio gestionado remoto vive entre uno y quince milisegundos según la región. Si el endpoint autoriza a nivel de objeto en una lista paginada de cincuenta elementos, ese coste se multiplica por cincuenta y puede dominar la latencia total de la API. Los proveedores serios ofrecen batch APIs y lookup inverso ("dame los objetos que este sujeto puede leer") para mitigarlo.

La residencia de datos es el segundo punto crítico. Subir relaciones que incluyen identificadores de usuarios y documentos a un proveedor SaaS tiene implicaciones RGPD en Europa, HIPAA en Estados Unidos y normativas locales en sectores regulados. Varios proveedores ofrecen planes dedicated o single-tenant en regiones específicas; otros solo operan en multi-tenant global. Hay que leer el contrato antes de comprometerse.

El lock-in es inevitable hasta cierto punto. Aunque AuthZed, Cerbos y OpenFGA tienen componentes open source, migrar el corpus de políticas y relaciones entre proveedores requiere reescribir en el DSL destino y, si el modelo es distinto (ABAC vs ReBAC), rediseñar la semántica. Una estrategia prudente es separar claramente el PEP (Policy Enforcement Point) de la aplicación mediante una capa de abstracción interna, de modo que cambiar de proveedor no obligue a tocar cada controlador.

> Un contrato de abstracción sobre el SDK del proveedor añade trabajo hoy pero permite migrar o descentralizar cuando la relación comercial cambia. El coste de no tenerlo se paga siempre más tarde.
{: .prompt-tip }

## Cuándo sí

EaaS encaja bien cuando hay muchas aplicaciones con modelos de permisos parecidos, equipos de producto que necesitan cambiar reglas sin pasar por despliegues de ingeniería y un apetito por gobernar la autorización centralmente. Una empresa con veinte productos internos y tres externos, cada uno con su esquema de roles y permisos, se beneficia enormemente de un único control plane donde compliance puede buscar "quién puede ver datos de salud" y obtener una respuesta consolidada.

También conviene cuando la velocidad de entrega importa más que el control absoluto del stack. Un equipo pequeño con varias SaaS B2B puede entregar su MVP antes si conecta Cerbos o Permit.io que si monta OPA con sus propios pipelines. El coste mensual compensa los meses de ingeniería de plataforma que ahorra.

## Cuándo no

En el extremo opuesto, una startup con una sola aplicación y un modelo de permisos simple (admin, member, viewer) no necesita EaaS. Un policy-as-code ligero en el propio repositorio cubre el caso y evita la dependencia de un tercero. Tampoco encaja cuando las regulaciones de residencia de datos imposibilitan enviar identificadores fuera de un perímetro concreto y el proveedor no ofrece despliegue dedicado en esa región.

El caso más complicado son las latencias sub-milisegundo estrictas, habituales en motores de búsqueda, feeds personalizados o trading. Ahí el PDP remoto es inviable y, aun con PDP local, la sobrecarga del SDK puede ser demasiado. La solución suele ser precalcular listas de autorización en el backend de datos y dejar el motor EaaS para cambios menos frecuentes.

## Encaje arquitectónico

EaaS no reemplaza a [RBAC, ABAC y ReBAC](/posts/RBAC-ABAC-ReBAC/); los implementa. Se sitúa entre la aplicación y el motor de decisión, absorbiendo el ciclo de vida de políticas y el gobierno. Donde antes había un debate entre OPA y OpenFGA, ahora hay un segundo eje: hacerlo uno mismo o delegar la operación. Esa es la pregunta real que EaaS obliga a responder.

## Artículos relacionados en esta serie

- [RBAC, ABAC y ReBAC](/posts/RBAC-ABAC-ReBAC/)
- [CASBIN, OPA y OpenFGA](/posts/CASBIN-OPA-OpenFGA/)
- [XACML](/posts/XACML/)
- [Autenticación y autorización](/posts/Autenticacion-y-Autorizacion/)
- [Zero Trust](/posts/Zero-Trust/)

## Referencias

- Pang et al. (2019). *Zanzibar: Google's Consistent, Global Authorization System*. USENIX ATC.
- AuthZed (2024). *SpiceDB Documentation*.
- Cerbos (2024). *Cerbos Policy Reference*.
- OpenFGA (2024). *OpenFGA Concepts and Documentation*.
- CNCF (2024). *Open Policy Agent Documentation*.
