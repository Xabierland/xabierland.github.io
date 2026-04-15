---
title: RBAC, ABAC y ReBAC: cómo evoluciona el modelo de autorización según la complejidad de tu sistema
author: Xabierland
description: >-
    RBAC, ABAC y ReBAC son los tres paradigmas que dominan la autorización moderna. Cada uno responde a un tipo distinto de complejidad organizativa y técnica. Elegir mal el modelo produce tablas de permisos inmanejables, auditorías imposibles o políticas que nadie sabe explicar.
date: 2026-05-27 10:00
categories: [DevSecOps, Seguridad de Aplicaciones]
tags: [RBAC, ABAC, ReBAC, Autorización, Control de Acceso, Arquitectura]
---

Una de las decisiones arquitectónicas que más consecuencias tiene a largo plazo es el modelo de autorización. Es una decisión silenciosa: rara vez aparece en una revisión de diseño con el peso que merece, pero condiciona años de desarrollo, el coste de cada cambio de permisos y la capacidad del negocio para crecer sin reescribir la plataforma. Este artículo forma parte de la serie [La evolución de la seguridad en aplicaciones modernas](/posts/La-evolucion-de-la-seguridad-en-aplicaciones-modernas/).

## Por qué tres modelos y no uno

La historia de la autorización es la historia de cómo la complejidad organizativa y la complejidad del dominio han ido rompiendo el modelo anterior. RBAC (Role-Based Access Control) nació cuando los sistemas tenían miles de usuarios y decenas de funciones de trabajo. ABAC (Attribute-Based Access Control) apareció cuando el contexto importaba tanto como el rol: hora del día, ubicación, nivel de clasificación del documento, proyecto al que está asignado. ReBAC (Relationship-Based Access Control) surgió cuando el dominio del problema era inherentemente un grafo: documentos compartidos, carpetas anidadas, equipos dentro de equipos, recursos propiedad de otros recursos.

No son modelos competidores que se sustituyan unos a otros. Son capas de expresividad distintas que resuelven problemas distintos. La mayoría de sistemas maduros acaban combinando dos o los tres, y el error arquitectónico más caro no es elegir el modelo equivocado, es insistir en que uno solo lo resuelve todo.

## RBAC: la base que nadie abandona del todo

RBAC se formalizó en 1992 en el trabajo de Ferraiolo y Kuhn en el NIST (National Institute of Standards and Technology) y terminó estandarizado como INCITS 359-2012. El modelo es conceptualmente limpio: los usuarios se asignan a roles, los roles agrupan permisos y el permiso es una dupla de acción sobre recurso. La decisión de acceso consulta si el rol efectivo del sujeto contiene el permiso requerido.

Lo que convirtió RBAC en el estándar de facto empresarial fue su alineamiento natural con la estructura organizativa. Un contable, un jefe de proyecto o un técnico de soporte son roles que ya existen en la empresa. Darles permisos como unidad es intuitivo, revisable en una auditoría y fácil de justificar ante un regulador.

### Jerarquías y separación de funciones

El estándar NIST distingue cuatro niveles: RBAC plano, RBAC jerárquico, RBAC con restricciones y RBAC simétrico. La jerarquía permite que un rol herede permisos de otro, lo que reduce duplicación. La separación de funciones o SoD (Separation of Duties) introduce restricciones del tipo "nadie puede ser a la vez quien aprueba un gasto y quien lo registra", algo que SOX y buena parte de la regulación financiera exige explícitamente. Estas restricciones se expresan como SoD estática (incompatibilidad en la asignación) o dinámica (incompatibilidad en una misma sesión).

### El límite real: la explosión de roles

El modelo se rompe cuando el negocio exige distinguir no por función sino por contexto. Aparecen roles como "jefe de proyecto de la región norte con acceso a documentos confidenciales del cliente X los días laborables". Cada nueva dimensión multiplica el número de roles: si hay diez proyectos, cinco regiones, tres niveles de confidencialidad y dos franjas horarias, se necesitan teóricamente trescientos roles para cubrir todas las combinaciones. En la práctica, las organizaciones acaban con miles de roles sintéticos, nombres crípticos y la sospecha fundada de que varios de ellos son equivalentes pero nadie se atreve a unificarlos.

> Si tu catálogo de roles tiene más entradas que empleados la empresa, el problema no es que RBAC se quede corto, es que alguien lo está usando para expresar atributos.
{: .prompt-warning }

## ABAC: cuando el contexto entra en la decisión

ABAC, formalizado por NIST en SP 800-162 (2014), modela la autorización como una función que recibe atributos del sujeto, del recurso, de la acción y del entorno, y devuelve una decisión. No hay roles privilegiados en el modelo: un rol es, si acaso, un atributo más del sujeto. La política se expresa como una regla lógica sobre esos atributos.

La expresividad es notablemente superior. Una regla como "un médico del mismo hospital que el paciente puede leer su historial durante su turno de guardia" se escribe directamente sin crear roles artificiales. El contexto (ubicación, turno, consentimiento del paciente) entra en la decisión sin doblar el modelo. Para entornos con clasificación multi-nivel, cumplimiento granular o reglas que cambian con frecuencia, ABAC es la respuesta natural.

### El coste oculto: auditoría y rendimiento

La expresividad tiene contrapartidas serias. La primera es la auditoría: una política ABAC puede ser formalmente correcta y a la vez incomprensible para un no-especialista. Responder a "¿quién puede acceder a este recurso?" ya no es consultar una tabla; es resolver un problema de satisfacibilidad sobre atributos potencialmente infinitos. Herramientas como los analizadores formales de políticas intentan mitigarlo, pero siguen siendo territorio de experto.

La segunda contrapartida es el rendimiento. Evaluar una política ABAC implica recolectar atributos desde varias fuentes (directorio, HR, clasificación del documento, señales de contexto), y cada llamada externa suma latencia al camino crítico. La caché agresiva de atributos ayuda, pero introduce el riesgo clásico de decisiones obsoletas: un empleado recién despedido sigue teniendo acceso hasta que el TTL (Time To Live) caduque.

## ReBAC: el grafo como unidad primaria

ReBAC llegó al mainstream con el paper de Google de 2019 sobre Zanzibar, el sistema de autorización que gobierna Drive, YouTube, Calendar y prácticamente todo el catálogo de consumo de Google. La idea central es elegante: la autorización se modela como un grafo de relaciones tipadas entre entidades. "user:alice es viewer de document:123", "group:engineering es parent de folder:backend", "document:123 está en folder:backend". La decisión de acceso se reduce a una consulta de alcanzabilidad en el grafo.

Para dominios naturalmente relacionales —documentos compartidos, carpetas anidadas, proyectos dentro de proyectos, multi-tenant jerárquico— ReBAC expresa en líneas lo que RBAC o ABAC exigirían contorsionar. Permisos heredados, reglas de compartición granular y propietarios efectivos dejan de ser políticas complicadas: son aristas del grafo.

### Consistencia: el aporte técnico de Zanzibar

El problema más sutil de ReBAC es la consistencia. Si un administrador revoca un permiso y un microsegundo después el usuario intenta una acción, ¿qué debe ver el sistema? Zanzibar introdujo los *zookies*, un token de consistencia que acompaña a cada escritura y permite a los lectores exigir una versión del grafo al menos tan reciente como el cambio observado. Sin esa primitiva, un sistema distribuido de autorización estaría condenado a race conditions donde el comportamiento depende de qué réplica responde.

Las implementaciones open source herederas de Zanzibar (OpenFGA, SpiceDB, Ory Keto) han hecho accesible este modelo a organizaciones sin la escala de Google. El precio a pagar es un nuevo servicio de autorización con su propia base de datos de relaciones, su propia semántica de consistencia y su propio contrato de latencia.

## Cuándo encaja cada uno

La pregunta útil no es cuál es el mejor modelo, es qué tipo de complejidad domina en tu sistema. RBAC encaja cuando los permisos se agrupan naturalmente por función de trabajo, los roles son estables en el tiempo y la granularidad contextual es baja. Aplicaciones de gestión internas, ERPs, herramientas administrativas y la práctica totalidad del software B2B tradicional viven cómodas en RBAC.

ABAC se impone cuando el acceso depende de condiciones dinámicas que no se pueden comprimir en un rol. Sanidad (reglas de consentimiento y turno), banca (límites por monto, hora y canal), defensa (clasificación y compartimentación) y cualquier dominio sujeto a regulación fina encuentran en ABAC el único modelo razonable.

ReBAC entra en juego cuando el dominio es un grafo: colaboración, compartición de documentos, recursos con jerarquía profunda, multi-tenant anidado, redes sociales. Todos los productos modelo "Google Docs" o "GitHub" acaban, de forma explícita o emergente, implementando un ReBAC. Intentar cubrir estos dominios con RBAC puro es lo que genera tablas de permisos de docenas de millones de filas y consultas que no caben en los SLO (Service Level Objective) razonables.

### Combinaciones en producción

La combinación es la norma, no la excepción. RBAC con restricciones ABAC es el patrón clásico: los roles definen la estructura base y los atributos imponen restricciones de contexto (horario, IP, nivel de riesgo). ReBAC con ABAC combina la expresividad del grafo con condiciones contextuales: "eres editor del documento, sí, pero solo desde la red corporativa". RBAC sobre ReBAC aparece cuando hay una estructura organizativa (roles) pero los recursos viven en un grafo (documentos y carpetas compartidas).

> Los modelos son aditivos. La pregunta no es cuál eliminar, es dónde aplicar cada uno para no pagar su coste donde no aporta.
{: .prompt-tip }

## Cómo encaja con el resto de la arquitectura

La elección de modelo condiciona directamente qué motor de política tiene sentido y cómo se despliega. [Motores de política como CASBIN, OPA y OpenFGA](/posts/CASBIN-OPA-OpenFGA/) reflejan esta diversidad: OPA destaca en ABAC/RBAC sobre configuración, OpenFGA nació para ReBAC y CASBIN cubre variantes de ambos con sus ficheros PERM. El estándar histórico [XACML](/posts/XACML/) formalizó ABAC antes de que el término fuera común y sigue vivo en grandes despliegues empresariales. Y cuando el PEP (Policy Enforcement Point) está en el API server de Kubernetes, los modelos se traducen a políticas de [admisión](/posts/Admission-Control-Kubernetes/) con su propia sintaxis pero la misma semántica de fondo.

El criterio práctico para decidir es sencillo de enunciar: si puedes explicar la política con una tabla de roles y permisos, RBAC basta; si necesitas una fórmula lógica sobre atributos, ABAC; si necesitas un grafo para representar quién posee o comparte qué, ReBAC. Y si la explicación incluye los tres, diseña para combinarlos desde el principio.

## Artículos relacionados en esta serie

- [CASBIN, OPA y OpenFGA](/posts/CASBIN-OPA-OpenFGA/)
- [XACML](/posts/XACML/)
- [Autenticación y autorización](/posts/Autenticacion-y-Autorizacion/)
- [Admission Control en Kubernetes](/posts/Admission-Control-Kubernetes/)

## Referencias

- Ferraiolo, D. F. y Kuhn, D. R. (1992). *Role-Based Access Control*. NIST.
- ANSI/INCITS (2012). *INCITS 359-2012 Role Based Access Control*.
- NIST SP 800-162 (2014). *Guide to Attribute Based Access Control (ABAC) Definition and Considerations*.
- Pang, R. et al. (2019). *Zanzibar: Google's Consistent, Global Authorization System*. USENIX ATC.
- OWASP (2024). *Authorization Cheat Sheet*.
