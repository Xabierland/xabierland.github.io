---
title: "Identity Governance and Administration: la capa de gobernanza sobre IAM"
author: Xabierland
description: >-
    IGA es la disciplina que responde a quién debería tener acceso, no solo quién lo tiene. Cubre el ciclo de vida completo de identidades y permisos, desde la solicitud hasta la recertificación. Es la capa que convierte un IdP operativo en un sistema auditable ante SOX, SOC 2, ISO 27001 o DORA.
date: 2026-12-09 10:00
categories: [DevSecOps, Seguridad de Aplicaciones]
tags: [IGA, Gobernanza, IAM, Cumplimiento, SoD, JML, Auditoria]
---

Este artículo forma parte de la serie [La evolución de la seguridad en aplicaciones modernas](/posts/La-evolucion-de-la-seguridad-en-aplicaciones-modernas/).

Un IdP bien configurado responde con precisión a la pregunta "¿puede Ana acceder al sistema de facturación ahora mismo?". Lo que no responde, al menos no sin ayuda, es "¿debería Ana tener ese acceso?", "¿cuándo se lo dieron y por qué?", "¿quién lo aprobó?", "¿sigue necesitándolo?", "¿entra en conflicto con otro permiso que también tiene?". Esas preguntas no son de autenticación ni de autorización en tiempo real: son de gobernanza, y viven en un plano distinto que se conoce como IGA, Identity Governance and Administration.

## La distinción IAM vs IGA

IAM (Identity and Access Management) opera en runtime: autentica usuarios, autoriza peticiones, gestiona sesiones. IGA opera en el plano de decisión y auditoría: gestiona el ciclo de vida de identidades y permisos, documenta por qué cada acceso existe, fuerza revisiones periódicas y detecta anomalías estructurales. IAM contesta en milisegundos; IGA trabaja en ciclos de días, semanas o trimestres.

Las dos capas están íntimamente conectadas pero su separación es útil. El IdP es la fuente de verdad operativa (aquí es donde [Keycloak, Okta y los IdP modernos](/posts/Keycloak-Okta-IdP/) viven), y el IGA es la fuente de verdad de intención: qué roles existen, qué permisos contiene cada uno, quién los pidió, quién los aprobó, cuándo caducan, cuándo se recertifican. Ambas se sincronizan, típicamente por [SCIM](/posts/SCIM/) o conectores propietarios, de modo que las decisiones tomadas en IGA terminan reflejándose en IdPs, directorios y aplicaciones downstream.

## Los procesos centrales

Una plataforma IGA organiza su trabajo alrededor de unos pocos procesos clásicos. El primero es Access Request, el flujo por el que un usuario solicita un permiso, el sistema identifica a los aprobadores correctos (su manager, el owner del recurso, un equipo de riesgo) y registra la aprobación como evidencia. Lo importante no es el formulario, es la trazabilidad: cada acceso que existe tiene un ticket con fecha, justificación y firma.

El segundo es Access Certification o Access Review, la revisión periódica donde managers y owners repasan los accesos de su equipo o su sistema y confirman, modifican o revocan. Cadencias típicas van de trimestral a anual según la criticidad del recurso. Una campaña de certificación mal diseñada se convierte en rubber stamping; una bien diseñada exige justificar accesos que el revisor no reconoce y cierra sola los que no se aprueban.

El tercero es Segregation of Duties (SoD), la detección de combinaciones de permisos incompatibles. El caso clásico es la misma persona pudiendo crear un proveedor y aprobar un pago, pero aparece en muchas variantes: el mismo usuario creando identidades y aprobando solicitudes, desplegando código y aprobando cambios, generando informes y modificando la fuente. Una política SoD es un conjunto de reglas que el IGA evalúa cada vez que se asigna un permiso y como revisión continua sobre el estado actual.

El cuarto es Role Mining, el descubrimiento de roles desde el uso real. Con años de permisos acumulados persona a persona, el modelo de roles idealizado en una hoja de Excel deja de parecerse al conjunto real. Herramientas de role mining analizan la matriz de usuarios por permisos, identifican clústeres (grupos de usuarios con permisos muy similares), y proponen roles candidatos que capturan la mayor parte del acceso real con menos asignaciones.

El quinto, probablemente el más visible, es Joiner/Mover/Leaver (JML): el tratamiento automatizado de altas, cambios de puesto y bajas. Un joiner nuevo recibe un paquete inicial según su puesto, un mover pierde accesos del puesto anterior y gana los del nuevo, un leaver pierde todo con garantía de que no queda huérfano.

## Anti-patrones que IGA resuelve

Cuando no hay IGA formal, ciertos anti-patrones emergen con fiabilidad. El role bloat: roles que nacieron pequeños y acumularon permisos para cubrir casos particulares, terminando como bolsas enormes que se asignan "porque es lo que todo el mundo tiene". La acumulación de privilegios: empleados que cambian de equipo varias veces y conservan los permisos de todos los puestos anteriores. Las orphan accounts: cuentas de servicio, buzones compartidos o usuarios externos que quedan tras bajas sin dueño claro. Los permisos por si acaso: accesos concedidos "por si algún día los necesita" que nunca se revisan.

Ninguno se resuelve con un IdP más potente. Se resuelven con procesos de revisión, con políticas que caducan accesos por inactividad, con owners designados para cada recurso, con evidencia ante auditoría. Eso es lo que da IGA.

> El métrica más útil para saber si una organización tiene IGA real es "porcentaje de accesos con justificación documentada y dueño identificado". Si esa cifra no se puede calcular, no hay IGA, hay hojas de cálculo.
{: .prompt-warning }

## Integración con el stack

Una plataforma IGA no vive aislada. Se integra con el HR como fuente autoritativa de identidades (workflows JML se disparan desde el HRIS), con el IdP como destino de cambios, con aplicaciones vía SCIM o conectores propietarios, con SIEM para correlación, con ticketing para flujos de aprobación. [SCIM](/posts/SCIM/) es el estándar de facto para el eje aplicaciones, aunque el conjunto de aplicaciones con SCIM maduro sigue siendo limitado; para el resto se recurre a conectores específicos (Active Directory, Salesforce, SAP, Workday, bases de datos) mantenidos por el fabricante del IGA.

La relación con [Privileged Access Management](/posts/Privileged-Access-Management/) es clara: el IGA gobierna el ciclo de vida de la identidad privilegiada (quién es administrador, cuándo caduca), el PAM gobierna cómo se usa ese privilegio en ejecución (sesión grabada, contraseña rotada, acceso just-in-time). Las dos capas son complementarias y las grandes plataformas ofrecen ambas o se integran estrechamente.

## Productos del mercado

El cuadrante de Gartner en IGA lleva años dominado por SailPoint, con IdentityIQ (on-premise, madurez histórica) e Identity Security Cloud (SaaS, más reciente). Saviynt compite directamente con una propuesta cloud-first y fuerte en regulación. Omada tiene presencia significativa en Europa. Oracle Identity Governance mantiene su base instalada en grandes corporaciones. ForgeRock (hoy parte de Ping) cubre la integración con su propio IdP. Microsoft ha entrado al mercado con Entra ID Governance, apalancando su presencia en Microsoft 365 para ofrecer access reviews, entitlement management y lifecycle workflows dentro del mismo tenant que ya usan las organizaciones.

La elección depende menos del motor de workflow y más del ecosistema de conectores, la capacidad de modelado (niveles de abstracción, políticas SoD complejas) y la experiencia de usuario para revisores, que es quien determina si las campañas de certificación funcionan o se convierten en teatro.

## Drivers regulatorios

IGA rara vez se implanta porque alguien decida que es buena idea. Se implanta porque un marco regulatorio lo exige, al menos implícitamente. SOX (Sarbanes-Oxley) demanda controles demostrables sobre acceso a sistemas financieros en empresas cotizadas en EE. UU. SOC 2 Type II evalúa esos controles durante un periodo. ISO 27001 incluye Anexo A con controles sobre gestión de accesos que sin IGA son imposibles de evidenciar a escala.

En la UE la presión ha aumentado recientemente. DORA (Digital Operational Resilience Act), aplicable desde enero de 2025 a entidades financieras, exige gestión documentada de accesos a sistemas críticos incluyendo controles SoD. NIS2, aplicable desde octubre de 2024 a sectores esenciales e importantes, incluye requisitos de control de acceso que, para organizaciones grandes, solo son realistas con una plataforma IGA. En el sector sanitario, HIPAA en EE. UU. y GDPR en la UE imponen el principio de mínimo privilegio con trazabilidad de accesos a datos personales sensibles.

## Cuándo hace falta IGA formal

El criterio práctico combina tamaño, regulación y complejidad. A partir de varios cientos de empleados con rotación normal, el JML manual se convierte en un agujero de seguridad garantizado: gente que se va y sigue teniendo acceso, gente que cambia de equipo y acumula permisos. A partir de auditorías externas periódicas, el coste de demostrar controles manualmente excede con diferencia el coste de una plataforma IGA. A partir de más de unas pocas decenas de aplicaciones integradas con el IdP, el volumen de entitlements hace ingobernable cualquier enfoque artesanal.

Cuando concurren los tres factores (escala, regulación y complejidad de aplicaciones), el debate no es si desplegar IGA sino cuál.

## Cuándo no

Una startup con veinte personas, un IdP SaaS, SCIM contra las cinco aplicaciones que usa, una revisión trimestral manual sobre una hoja de cálculo bien mantenida, y un proceso claro de baja, no necesita IGA. El coste de licencia y de operación de una plataforma formal excede su beneficio marginal. Lo mismo aplica a SaaS pequeños donde el modelo de permisos es plano y la plantilla estable.

El criterio invertido es útil: si hoy no puedes responder en una hora a "enséñame todos los accesos que tiene Juan Pérez y por qué", pero tampoco vas a recibir una auditoría que lo exija, el IGA puede esperar. El día que la pregunta sea inevitable, empezar a implantarlo es tarde pero posible.

> La transición típica no es "sin IGA" a "SailPoint completo". Es "sin IGA" a "Entra ID Governance o equivalente ligero" a "plataforma dedicada cuando la regulación lo fuerza".
{: .prompt-tip }

## Encaje en el ecosistema

IGA es la capa que convierte una infraestructura de identidad operativa en un sistema gobernado. Se apoya en [Autenticación y autorización](/posts/Autenticacion-y-Autorizacion/) como fundamento técnico, usa [SCIM](/posts/SCIM/) como transporte a aplicaciones, se integra con [Privileged Access Management](/posts/Privileged-Access-Management/) para el eje de privilegio, y alimenta los flujos de detección descritos en [ITDR](/posts/ITDR/). No aparece en la ruta crítica de cada petición HTTP, pero determina qué peticiones deberían existir en primer lugar.

## Artículos relacionados en esta serie

- [Autenticación y autorización](/posts/Autenticacion-y-Autorizacion/)
- [SCIM](/posts/SCIM/)
- [Privileged Access Management](/posts/Privileged-Access-Management/)
- [Keycloak, Okta y los IdP modernos](/posts/Keycloak-Okta-IdP/)
- [ITDR](/posts/ITDR/)

## Referencias

- Gartner (2024). *Magic Quadrant for Identity Governance and Administration*.
- ISO/IEC 27001 (2022). *Information security, cybersecurity and privacy protection. Information security management systems*.
- Parlamento Europeo (diciembre 2022). *Reglamento (UE) 2022/2554 sobre la resiliencia operativa digital del sector financiero (DORA)*.
- Parlamento Europeo (diciembre 2022). *Directiva (UE) 2022/2555 relativa a medidas para un elevado nivel común de ciberseguridad en la Unión (NIS2)*.
- NIST SP 800-162 (enero 2014). *Guide to Attribute Based Access Control (ABAC) Definition and Considerations*.
