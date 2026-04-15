---
title: "Motores de política: CASBIN, OPA y OpenFGA como cerebro de la autorización"
author: Xabierland
description: >-
    CASBIN, OPA y OpenFGA representan tres filosofías distintas de externalizar la lógica de autorización fuera del código de negocio. Cada uno responde a un modelo mental diferente y tiene un perfil operativo propio. Elegir entre ellos es menos una cuestión de funcionalidad que de cómo quieres que tu plataforma razone sobre permisos.
date: 2026-06-03 10:00
categories: [DevSecOps, Seguridad de Aplicaciones]
tags: [CASBIN, OPA, OpenFGA, Autorización, Policy Engine, Microservicios, CNCF]
---

La autorización dentro del código de negocio es deuda técnica con fecha de caducidad. En cuanto aparece el segundo servicio, el tercer cliente o la primera auditoría seria, la necesidad de externalizar las decisiones se vuelve evidente. Este artículo forma parte de la serie [La evolución de la seguridad en aplicaciones modernas](/posts/La-evolucion-de-la-seguridad-en-aplicaciones-modernas/).

## Por qué externalizar la autorización

La razón estructural para sacar la autorización del código de negocio es la misma que empuja a externalizar cualquier política transversal: centralizar lo que cambia por motivos distintos al resto del código. Las reglas de acceso evolucionan por requisitos regulatorios, decisiones de producto y eventos de seguridad; nada de eso debe obligar a recompilar el dominio.

El argumento económico es todavía más contundente. Una organización con diez servicios implementando sus propias comprobaciones de permisos tiene diez implementaciones que auditar, diez conjuntos de tests que mantener y diez puntos donde un refactor puede introducir un fallo explotable. Un motor de política centralizado convierte esa topología en un único cerebro consultable, versionado y con una semántica uniforme.

Los tres motores que dominan el ecosistema actual —CASBIN, OPA (Open Policy Agent) y OpenFGA— responden a esta necesidad con estrategias muy distintas. No son intercambiables: cada uno encarna un modelo mental, un contrato operativo y una huella de despliegue particulares.

## CASBIN: biblioteca embebida y configuración PERM

CASBIN nació en 2017 como una biblioteca en Go (con ports oficiales a Java, Node, Python, .NET, PHP y Rust) que formaliza la autorización mediante el metamodelo PERM: Policy, Effect, Request, Matchers. Cada uno es una sección en un archivo de configuración que describe, respectivamente, qué forma tienen las políticas, cómo se combinan los efectos (permit-override, deny-override, priority), qué atributos trae la solicitud y cómo se comparan políticas y solicitud.

Esta abstracción permite que el mismo motor implemente RBAC puro, RBAC jerárquico, ABAC, control de acceso sobre recursos RESTful e incluso modelos más exóticos, cambiando solo el fichero de configuración. Las políticas en sí se almacenan en adaptadores persistentes: base de datos relacional, Redis, fichero plano o cualquier backend para el que exista driver.

### Dónde brilla

CASBIN encaja cuando se necesita embeber la autorización dentro del proceso y se quiere flexibilidad de modelo sin desplegar un sidecar. Monolitos que no pueden pagar el coste de una llamada de red por cada decisión, bibliotecas de terceros que incorporan autorización como extensión o aplicaciones Go donde el motor es literalmente otro paquete importado son su territorio natural. La latencia de decisión es la de una llamada a función, del orden de microsegundos.

### Sus límites

El modelo PERM es potente pero opaco para quien no lo domina. Depurar una política que combina múltiples matchers con *RoleManager* personalizado no es trivial. La distribución de políticas requiere una estrategia explícita (watcher sobre Redis, replicación desde base de datos) y la coherencia entre réplicas cuando el pool de instancias es grande exige cuidado. Y al ser biblioteca, cada lenguaje tiene su propio port con un ritmo de evolución no siempre sincronizado.

## OPA: policy-as-code con Rego

OPA apareció en 2016 y alcanzó el estatus de CNCF Graduated en 2021, una señal clara de madurez en el ecosistema cloud-native. Su propuesta es distinta: un daemon o sidecar expone una API HTTP sobre la que se evalúan políticas escritas en Rego, un lenguaje declarativo inspirado en Datalog. La política se publica al agente como *bundle* (un tarball firmado con reglas y datos), el agente la evalúa localmente y devuelve una decisión.

Rego es el elemento más distintivo y también el más divisivo. Su sintaxis es elegante para quien viene de lógica declarativa y desconcertante para quien espera control de flujo imperativo. La compensación es notable: una política Rego puede consultarse parcialmente (partial evaluation), lo que permite compilarla a una query SQL o a una condición en otro lenguaje, habilitando patrones de autorización sobre listas enteras sin pagar una llamada por fila.

### Donde gana terreno

El caso de uso donde OPA no tiene rival es la política de plataforma. Admisión en [Kubernetes vía Gatekeeper](/posts/Admission-Control-Kubernetes/), control de configuración en Terraform, validación de manifests de Envoy, autorización en service mesh: todo ello comparte la misma primitiva —un JSON de entrada, una política, una decisión— y OPA está diseñado exactamente para eso.

Las capacidades operativas son maduras: distribución por bundles, *decision logs* que registran cada evaluación para auditoría, hot reload sin reiniciar, telemetría vía Prometheus y soporte para descubrimiento dinámico. En despliegue sidecar, la latencia típica es inferior al milisegundo, lo que lo hace viable en el camino crítico de servicios de alto throughput.

### Sus puntos débiles

OPA no es un motor de relaciones. Modelar "¿alice puede editar document:123?" cuando la respuesta depende de transitar un grafo de diez niveles es técnicamente posible pero operativamente costoso: cargar el grafo como datos en cada evaluación escala mal y las queries sobre miles de relaciones se vuelven lentas. Rego es precioso para ABAC y para políticas sobre configuración, pero para ReBAC nativo hay mejores herramientas.

## OpenFGA: Zanzibar para el resto del mundo

OpenFGA fue iniciado por Auth0 (ahora Okta) en 2022 y aceptado como proyecto CNCF Sandbox poco después. Es una implementación abierta del modelo de Google Zanzibar, con un modelo de relaciones tipadas definidas en un DSL (Domain Specific Language) propio y un servicio independiente que almacena las *relationship tuples* en una base de datos.

La expresividad de OpenFGA es la de un sistema de relaciones: se definen tipos (document, folder, user, group) y relaciones entre ellos (viewer, editor, owner, parent), con reglas de *userset rewrite* que permiten, por ejemplo, que "editor de un documento es automáticamente viewer" o que "miembro del grupo propietario de una carpeta es viewer de todos los documentos dentro de ella". La consulta típica —*Check*— responde a "¿es el sujeto X relacionado mediante Y con el objeto Z?" recorriendo el grafo por detrás.

### Consistencia y modelo operativo

Como Zanzibar, OpenFGA introduce un token de consistencia que permite al cliente exigir una lectura al menos tan reciente como una escritura previa. Esto es crítico para evitar que una revocación de permiso quede invisible a un lector en otra réplica. El motor expone API gRPC y HTTP, un Playground para modelar y un conjunto de SDKs en los lenguajes habituales.

### Cuándo es la respuesta

OpenFGA gana cuando el dominio es un grafo. Productos tipo Drive, GitHub, Notion, Figma o cualquier SaaS multi-tenant con compartición granular encuentran en OpenFGA la expresión natural de su modelo. Lo que en OPA requeriría cargar estructuras sintéticas, en OpenFGA es una lista de tuplas directa.

Su coste es un servicio más en el inventario operativo, con su propia base de datos, sus propios backups, su latencia de Check bajo SLO y el cuidado de mantener el modelo de tipos versionado junto al código.

## Matriz de decisión

La elección entre los tres se puede enmarcar sobre cuatro ejes. Por **tipo de política**, CASBIN y OPA destacan en lógica sobre atributos y configuración; OpenFGA en relaciones sobre un grafo. Por **modelo de despliegue**, CASBIN es biblioteca embebida; OPA es daemon local (sidecar) o remoto; OpenFGA es servicio centralizado con su propia persistencia. Por **latencia de decisión**, CASBIN gana al ser in-process, OPA es submilisegundo como sidecar, OpenFGA añade el coste de red y la consulta sobre el grafo. Por **ecosistema**, OPA domina Kubernetes y plataformas, OpenFGA domina producto SaaS, CASBIN domina aplicaciones que embeben autorización como feature.

> La pregunta útil no es cuál es mejor, es qué estás modelando: reglas sobre atributos (OPA/CASBIN) o relaciones en un grafo (OpenFGA).
{: .prompt-tip }

## Patrones de integración

El patrón más extendido en arquitecturas cloud-native es **OPA como sidecar** junto a cada servicio o en cada nodo del service mesh. La ventaja es latencia baja y disponibilidad desacoplada del plano de control: si el distribuidor de políticas cae, el sidecar sigue sirviendo la última versión. El coste es densidad de procesos y consumo agregado de memoria no despreciable en clústeres grandes.

El **servicio centralizado** es la norma en OpenFGA y en despliegues OPA que prefieren un único punto de evaluación. Simplifica versionado y observabilidad a cambio de introducir un hop de red en el camino crítico y un dominio de fallo compartido.

La **biblioteca embebida** (CASBIN, OPA como librería Go) es adecuada cuando la aplicación no puede tolerar latencia de red o cuando el número de procesos no justifica un sidecar. Exige estrategia de distribución de políticas y cuidado con el arranque en frío.

### Consideraciones operativas

Tres decisiones operativas dominan cualquier despliegue. La **latencia de decisión** como SLO explícito: si el servicio responde en P99 bajo 50 ms, la autorización no puede consumir la mitad. La **disponibilidad** del motor: una decisión que no se puede evaluar debe ser fail-closed en producción y eso exige cachés locales bien pensadas. La **invalidación de caché** cuando cambian políticas o atributos: un token o una relación caducados son el origen clásico de accesos indebidos post-revocación.

## Cómo encaja con el resto de la arquitectura

Los motores no existen en el vacío. Implementan los modelos de [RBAC, ABAC y ReBAC](/posts/RBAC-ABAC-ReBAC/) sobre las primitivas del estándar conceptual de [XACML](/posts/XACML/), con el motor como PDP y el servicio consumidor como PEP. En Kubernetes, OPA se convierte en el [controlador de admisión](/posts/Admission-Control-Kubernetes/) con Gatekeeper; en un [API Gateway o Service Mesh](/posts/API-Gateway-BFF-Service-Mesh-Sidecar/) actúa como decisor externo invocado por el proxy. La elección correcta del motor es consecuencia de haber entendido primero el modelo: cuando el modelo está claro, el motor suele ser evidente.

## Artículos relacionados en esta serie

- [RBAC, ABAC y ReBAC](/posts/RBAC-ABAC-ReBAC/)
- [XACML](/posts/XACML/)
- [Admission Control en Kubernetes](/posts/Admission-Control-Kubernetes/)
- [API Gateway, BFF y Service Mesh](/posts/API-Gateway-BFF-Service-Mesh-Sidecar/)

## Referencias

- CNCF (2024). *Open Policy Agent Documentation*.
- CASBIN (2024). *Documentation: PERM models*.
- Auth0/Okta (2022). *OpenFGA Documentation: Zanzibar-inspired authorization*.
- Pang, R. et al. (2019). *Zanzibar: Google's Consistent, Global Authorization System*. USENIX ATC.
- CNCF (2024). *CNCF Landscape: Security & Compliance — Policy*.
