---
title: "Admission Control en Kubernetes: cómo OPA Gatekeeper y Kyverno policían tu clúster"
author: Xabierland
description: >-
    El control de admisión es la última línea de defensa antes de que un objeto se persista en Kubernetes. OPA Gatekeeper y Kyverno compiten por este espacio con filosofías opuestas: Rego frente a YAML nativo. La elección condiciona la ergonomía diaria del equipo de plataforma y la robustez de la postura de seguridad.
date: 2026-08-19 10:00
categories: [DevSecOps, Seguridad de Aplicaciones]
tags: [Admission Control, OPA Gatekeeper, Kyverno, Kubernetes, Policy, DevSecOps]
---

El API server de Kubernetes es el único punto por el que cualquier cambio al estado deseado del clúster tiene que pasar, y el control de admisión es el momento exacto en que la plataforma puede decir "este manifiesto no entra" o "entra, pero no así". Este artículo forma parte de la serie [La evolución de la seguridad en aplicaciones modernas](/posts/La-evolucion-de-la-seguridad-en-aplicaciones-modernas/).

## Por qué existe el control de admisión

Kubernetes separa el control de acceso en dos preguntas distintas. La primera, resuelta por RBAC, es si el usuario o service account tiene permiso para ejecutar el verbo sobre el recurso: puedo crear pods en el namespace payments. La segunda, resuelta por admisión, es si el objeto concreto que pretende crearse cumple las políticas de la plataforma: el pod declara un límite de memoria, no se ejecuta como root, su imagen viene de un registro aprobado y tiene las etiquetas exigidas por el sistema de costes.

Confiar esta segunda capa solo a RBAC es imposible. RBAC habla de verbos y recursos, no del contenido de los manifiestos. Por eso el pipeline del API server separa, tras la autenticación y la autorización RBAC, una fase específica que inspecciona el objeto, puede modificarlo (mutating) y puede rechazarlo (validating) antes de que se persista en etcd.

Sin admisión, la única forma de imponer políticas de plataforma sería hacerlo en el CI/CD, con todos los problemas asociados: los caminos alternativos (kubectl directo desde un portátil con credenciales), los controladores que crean objetos dinámicamente (Horizontal Pod Autoscaler, operadores) y las escaladas de privilegio donde alguien con permisos de edición bypaseaba la validación del pipeline.

## El pipeline del API server

El orden canónico es authN, authZ (RBAC), admisión mutating, validación de esquema, admisión validating, persistencia. Los controladores in-tree históricos (ServiceAccount, ResourceQuota, LimitRanger, NamespaceLifecycle) vienen compilados con el API server y cubren funcionalidades básicas. Entre ellos, el más relevante hoy es **PodSecurityAdmission** (PSA), GA desde Kubernetes 1.25, que sustituyó a las PodSecurityPolicies (PSP) deprecadas en 1.21 y eliminadas en 1.25.

PSA aplica los **Pod Security Standards** (Privileged, Baseline, Restricted) a nivel de namespace mediante etiquetas, con modos enforce, audit y warn. Es un buen baseline pero cubre solo seguridad de pods y no es extensible; para cualquier política que salga de ese dominio hace falta otra cosa.

### Webhooks dinámicos

Kubernetes ofrece dos tipos de webhook dinámico: **MutatingAdmissionWebhook**, que puede modificar el objeto (inyectar sidecars, añadir etiquetas, establecer defaults), y **ValidatingAdmissionWebhook**, que solo puede aceptar o rechazar. El mutating corre antes para que el validating vea el objeto final. Ambos son HTTPS a un servicio externo, con el inherente coste de red y los peligros operativos de cualquier dependencia en el camino crítico del API server.

### ValidatingAdmissionPolicy en el árbol

Desde Kubernetes 1.30 (GA) existe **ValidatingAdmissionPolicy**, una alternativa in-tree basada en CEL (Common Expression Language). Sus políticas se evalúan dentro del API server sin hop de red, lo que elimina la dependencia externa y reduce latencia. Para políticas sencillas y de alto volumen es una opción atractiva; sin embargo, su poder expresivo es menor que el de Rego o de un motor completo, y aún no cubre mutaciones (la contraparte MutatingAdmissionPolicy está en camino pero en alpha en las versiones recientes).

## OPA Gatekeeper: Rego como estándar de facto

OPA Gatekeeper lleva a Kubernetes la fuerza del motor OPA (Open Policy Agent) mediante un **Constraint Framework** de dos niveles. Un *ConstraintTemplate* es una CRD (Custom Resource Definition) que define una política parametrizable escrita en Rego; un *Constraint* es una instancia concreta con parámetros específicos aplicada a un conjunto de recursos. Este patrón permite que el equipo de seguridad escriba una plantilla "requiere etiqueta X" y cada aplicación declare su constraint con sus etiquetas obligatorias.

Gatekeeper integra modo **audit**, que evalúa los objetos ya existentes periódicamente y reporta violaciones sin bloquearlas, útil cuando se introduce una política nueva sobre un clúster con estado previo. Su **soporte de mutación** permite corregir objetos (añadir defaults, normalizar etiquetas) con la misma infraestructura de constraints.

### Fortalezas y fricciones

Rego es expresivo y ya conocido por cualquier equipo que use OPA en otros contextos. La misma política de "imágenes solo desde registro aprobado" puede vivir en Gatekeeper para admisión Kubernetes, en un sidecar OPA para autorización de servicios y en un bundle OPA para validar configuración Terraform. Esa uniformidad de lenguaje es el argumento más fuerte para Gatekeeper en organizaciones ya invertidas en OPA.

La fricción es la curva de aprendizaje. Rego no se parece a nada que la mayoría de equipos de plataforma haya escrito, y depurar una política que no funciona exige familiaridad con sus particularidades (variables, unificación, evaluación perezosa). Los logs de Gatekeeper y el *constraint status* ayudan, pero la barrera inicial es real.

## Kyverno: policy-as-YAML nativo

Kyverno, CNCF Incubating (graduado a estatus Graduated en 2024), apuesta por una filosofía opuesta: las políticas son CRDs YAML con la misma forma que cualquier otro manifiesto Kubernetes. Quien sabe escribir un Deployment sabe leer una policy Kyverno, y el editor con autocompletado YAML que ya usa funciona igual para ambos. Esta ergonomía es lo que ha hecho crecer a Kyverno rápidamente en entornos donde el equipo de plataforma no quiere añadir un lenguaje nuevo al inventario.

Kyverno cubre **validate**, **mutate**, **generate** (crea recursos derivados, como una NetworkPolicy por namespace) y **cleanup** (elimina recursos según una política de retención). El **image verification** integra Cosign para verificar firmas y attestations SLSA (Supply-chain Levels for Software Artifacts) directamente desde la policy, algo que con Gatekeeper requiere ejercicios de integración más verbosos.

### Dónde se diferencian

Kyverno es menos expresivo que Rego para políticas complejas con lógica arbitraria. Cuando la regla es "si el namespace tiene esta anotación, exige estos labels, excepto para estos casos especiales", Rego lo expresa con claridad y Kyverno necesita encadenar patrones de match/exclude que se vuelven largos. A cambio, el noventa por ciento de las políticas reales son "exige esta etiqueta", "prohíbe este campo", "inyecta este sidecar" y en ese territorio Kyverno es demostrablemente más directo.

La comunidad Kyverno mantiene un catálogo extenso de políticas listas para usar (CIS Benchmarks, Pod Security Standards, best practices), lo que acelera la adopción en clústeres nuevos.

## Comparación pragmática

La elección entre Gatekeeper y Kyverno rara vez se decide por capacidades puras. Se decide por tres factores. **Competencia del equipo**: si ya se usa OPA en otros lugares, Gatekeeper capitaliza esa inversión; si el equipo vive en YAML y Helm, Kyverno reduce fricción. **Sofisticación de las políticas**: políticas con lógica condicional compleja encajan mejor en Rego; políticas declarativas de match/validate encajan mejor en Kyverno. **Necesidad de image verification y generación**: Kyverno cubre estos casos de forma nativa; Gatekeeper necesita complementos.

ValidatingAdmissionPolicy (CEL) no sustituye a ninguno de los dos todavía, pero desplaza el baseline: políticas simples que antes justificaban un webhook ahora pueden evaluarse in-tree, lo que reduce la carga sobre Gatekeeper/Kyverno para casos triviales.

> Añadir un webhook al API server es añadir una dependencia a la ruta crítica de cada kubectl apply. Trátalo como lo que es: un SLO compartido con la plataforma.
{: .prompt-warning }

## Consideraciones operativas

El modo de fallo es la decisión más consecuente. Un webhook configurado como **failurePolicy: Fail** convierte una caída del servicio en parálisis del clúster: no entran deployments nuevos, los operadores fallan al reconciliar y las regresiones se agravan. Un **failurePolicy: Ignore** permite que el clúster siga funcionando a costa de bypasear las políticas durante la caída. La elección correcta depende del recurso: bloquear pods sin recursos declarados puede ser tolerante (Ignore), bloquear pods privilegiados no lo es (Fail).

El rendimiento en clústeres grandes es otro eje. Cada webhook añade una llamada HTTPS al API server por cada create/update en su scope. Con miles de objetos por minuto y docenas de webhooks, la latencia agregada del API server se degrada. Las técnicas conocidas son limitar el scope con *objectSelector* y *namespaceSelector*, pre-filtrar agresivamente antes de evaluar la política, dimensionar los controladores con HPA y monitorizar la latencia P99 de cada webhook como métrica de primer nivel.

Los **mecanismos de excepción** son inevitables. Ninguna política sobrevive un mes en producción sin casos legítimos que necesiten excepción: una migración temporal, un sistema legacy, un entorno de investigación. Gatekeeper y Kyverno ofrecen anotaciones y selectores para esto; la disciplina es versionar las excepciones como código, darles fecha de caducidad y auditar su uso.

### Políticas típicas

El catálogo mínimo que cualquier clúster serio acaba implementando: etiquetas obligatorias (owner, cost-center, data-classification), prohibición de pods privilegiados o con hostPath arbitrario, aplicación del nivel PSS correspondiente al namespace, registros de imagen en una lista blanca, verificación de firmas para producción, límites de recursos declarados, y NetworkPolicies mínimas generadas automáticamente. Cada uno de estos ataja una clase entera de incidentes.

### Supply chain como extensión natural

El ángulo de supply chain (verificación de firmas Cosign, attestations SLSA, generación de SBOMs asociados al despliegue) encaja de forma natural en la admisión. El pod no entra si la imagen no está firmada por una clave autorizada; el deployment no se actualiza si el registry no tiene attestation válida del build pipeline. Kyverno tiene aquí una ventaja de ergonomía notable y la comunidad Sigstore lo ha convertido en su vector de adopción principal.

## Cómo encaja con el resto de la arquitectura

El control de admisión es literalmente el PEP/PDP del [modelo XACML](/posts/XACML/) aplicado a Kubernetes: el API server es el PEP y el webhook (Gatekeeper o Kyverno) es el PDP. El motor subyacente en Gatekeeper es el mismo [OPA](/posts/CASBIN-OPA-OpenFGA/) que puede usarse para autorización aplicativa. Los [modelos RBAC/ABAC/ReBAC](/posts/RBAC-ABAC-ReBAC/) viven también aquí: RBAC nativo para verbos sobre recursos, ABAC vía constraints sobre atributos del manifiesto. Y toda esta capa es necesaria pero insuficiente: la admisión protege el momento de crear el objeto, pero no la conducta en tiempo de ejecución. Para eso hace falta NetworkPolicy, autorización en service mesh y runtime security; es decir, todo lo que [Zero Trust](/posts/Zero-Trust/) exige.

## Artículos relacionados en esta serie

- [CASBIN, OPA y OpenFGA](/posts/CASBIN-OPA-OpenFGA/)
- [XACML](/posts/XACML/)
- [RBAC, ABAC y ReBAC](/posts/RBAC-ABAC-ReBAC/)
- [Zero Trust](/posts/Zero-Trust/)

## Referencias

- Kubernetes (2024). *Documentation: Admission Controllers Reference*.
- Kubernetes (2024). *KEP-3488: CEL-based Admission Control (ValidatingAdmissionPolicy GA en 1.30)*.
- Open Policy Agent (2024). *OPA Gatekeeper Documentation y Constraint Framework*.
- CNCF (2024). *Kyverno Documentation*.
- NIST SP 800-190 (2017). *Application Container Security Guide*.
- SLSA (2023). *Supply-chain Levels for Software Artifacts Framework*.
