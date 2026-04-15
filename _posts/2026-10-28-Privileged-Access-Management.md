---
title: "Privileged Access Management: acceso efímero, just-in-time y break-glass"
author: Xabierland
description: >-
    PAM (Privileged Access Management) es la disciplina que gobierna cómo se concede, supervisa y revoca el acceso a sistemas críticos. Resuelve el problema de las cuentas de administración compartidas, las credenciales permanentes y los accesos sin trazabilidad. En el ecosistema moderno, PAM converge con Zero Trust, Vault y los IdP para ofrecer credenciales efímeras y sesiones auditadas.
date: 2026-10-28 10:00
categories: [DevSecOps, Seguridad de Aplicaciones]
tags: [PAM, Zero Trust, Vault, Teleport, Boundary, JIT, Break-glass]
---

Este artículo forma parte de la serie [La evolución de la seguridad en aplicaciones modernas](/posts/La-evolucion-de-la-seguridad-en-aplicaciones-modernas/).

Si alguien tiene acceso permanente a producción con una cuenta de administrador compartida, el incidente no es una cuestión de si ocurrirá, sino de cuándo. PAM (Privileged Access Management) nació para cerrar esa brecha: mover el acceso privilegiado de un modelo de "cuenta estática con contraseña en un gestor" a un modelo donde la credencial se genera en el momento, dura minutos, queda registrada y exige justificación.

## Por qué el modelo clásico se rompió

Durante décadas, el acceso privilegiado se gestionó con cuentas nominadas (o peor, compartidas) que tenían permisos permanentes sobre la infraestructura. El DBA tenía su usuario en el motor de base de datos, el SRE tenía su llave SSH en los bastiones, el auditor recibía un acceso de solo lectura que nadie revocaba al terminar la auditoría. El resultado se puede enumerar: credenciales filtradas por ingeniería social, cuentas huérfanas de empleados que ya no estaban, sudo sin trazabilidad real, contraseñas de root rotadas "cuando toque" y un cumplimiento normativo que vivía más en las hojas de cálculo que en los sistemas.

El problema de fondo es que el privilegio se modelaba como un atributo estable del usuario, cuando en realidad es un atributo contextual de una tarea concreta. Un SRE no necesita root en el clúster 24x7; necesita root durante los quince minutos que dura un incidente. Un DBA no necesita acceso de escritura a la base de datos de producción; lo necesita para ejecutar una migración concreta el martes a las 21:00. PAM se construye sobre esa observación.

## Cómo funciona PAM en la práctica

La idea central es que el acceso se solicita, se aprueba y se concede con una ventana temporal corta. Cuando el ingeniero necesita entrar a un servidor, no usa una llave guardada en su laptop; abre el portal de PAM, selecciona el recurso, escribe una justificación ("incidente INC-4821, investigar latencia en el servicio de pagos") y el sistema genera una credencial efímera, normalmente un certificado SSH firmado por una CA interna con un TTL de entre quince minutos y unas horas.

Esa sesión pasa siempre por un proxy intermedio. El proxy autentica al usuario contra el IdP corporativo (con MFA y, cuando el recurso es sensible, un step-up adicional), registra cada comando que el usuario ejecuta, graba la sesión completa si es un terminal interactivo y la termina automáticamente al expirar la ventana. El usuario nunca ve la contraseña real de root ni la clave privada del host; el proxy los tiene, él no.

Sobre este esqueleto se apilan dos patrones. El primero es JIT (Just-in-Time) access: el usuario no tiene el permiso asignado en su perfil de forma permanente, sino que se le eleva temporalmente tras la aprobación. El segundo es la aprobación por flujo de trabajo: dependiendo de la criticidad, el acceso requiere la firma de un mánager, de un compañero de guardia o incluso de dos personas en modo "cuatro ojos" para operaciones destructivas.

> Diseñar el workflow de aprobación es tan importante como el producto PAM. Si aprobar un acceso de emergencia toma cuarenta minutos porque el mánager está en otra zona horaria, el equipo creará atajos y el control se convierte en teatro de seguridad.
{: .prompt-warning }

## Break-glass: cuando todo lo demás falla

Por muy bien que se diseñe el flujo, hay situaciones en las que la automatización no puede esperar: el IdP está caído, el proveedor cloud ha perdido conectividad con el PAM o una brecha activa exige acceso inmediato al panel de control. Para esos casos existe el acceso de emergencia, conocido como break-glass.

Un flujo de break-glass bien construido tiene tres propiedades. La credencial de emergencia vive sellada (a menudo en un HSM o en un sobre físico en una caja fuerte con cámara), su uso dispara alertas inmediatas a múltiples canales (PagerDuty, canal ejecutivo, correo al CISO), y la sesión que se abre con ella queda marcada como "emergencia" en la auditoría para revisión obligatoria posterior. La paradoja sana del break-glass es que debe ser fácil de usar en crisis y difícil de usar frívolamente: se consigue con fricción social y trazabilidad, no con fricción técnica.

## El catálogo actual

El espacio PAM ha vivido una transformación profunda. Los incumbentes tradicionales, con CyberArk a la cabeza, venían del mundo on-premise de gestión de credenciales y bóvedas de contraseñas, muy enfocados a cumplir PCI DSS y SOX. Siguen dominando la banca y el sector público, pero su arquitectura pesa cuando se trabaja con infraestructura efímera en la nube.

La generación cloud-native creció alrededor de tres propuestas. HashiCorp Boundary se integra nativamente con Vault, usa identidades Workload desde SPIFFE para los agentes y modela el acceso como un grafo de hosts y servicios. Teleport ofrece proxies SSH, Kubernetes, bases de datos y RDP con grabación de sesión y certificados efímeros firmados por su propia CA. StrongDM construye una malla de acceso que abstrae protocolos: el usuario final se conecta al proxy local y este se encarga de traducir al protocolo nativo de cada recurso.

En el mundo puramente cloud, AWS IAM Identity Center (antes AWS SSO) con sesiones federadas temporales, junto con servicios como AWS Systems Manager Session Manager, han absorbido buena parte del caso de uso que antes requería un bastion host clásico. Azure Privileged Identity Management (PIM) hace algo equivalente para el ecosistema Microsoft, elevando roles de Entra ID durante ventanas acotadas con aprobación.

## Encaje con el resto del stack

PAM no vive aislado. Vault aporta las credenciales dinámicas de base de datos y de servicios cloud que PAM distribuye; [SPIFFE y SPIRE](/posts/SPIFFE-SPIRE/) proporcionan la identidad de los agentes PAM desplegados en los nodos; el IdP corporativo autentica al humano y aplica [MFA](/posts/MFA/) con step-up; la [PKI](/posts/PKI/) interna firma los certificados SSH o X.509 de corta duración que viajan por el proxy. El resultado es un sistema donde cada pieza tiene una responsabilidad acotada y ninguna guarda credenciales eternas.

La relación con [Zero Trust](/posts/Zero-Trust/) es directa: PAM materializa dos principios clave del modelo, la verificación continua (cada sesión se reautentica, cada elevación se autoriza explícitamente) y el mínimo privilegio efectivo (el permiso solo existe durante la tarea). Sin PAM, Zero Trust sobre infraestructura queda en papel mojado.

> Una buena señal de madurez de un programa PAM es que el equipo de plataforma no sepa la contraseña de root de ningún host de producción. Si alguien puede recitarla, algo falla.
{: .prompt-tip }

## Cuándo conviene, cuándo estorba

PAM brilla cuando hay escala (decenas de ingenieros accediendo a cientos o miles de recursos), regulación (auditoría, trazabilidad, segregación de funciones) o superficie crítica (datos financieros, sanitarios, infraestructura esencial). En esos contextos, el coste de desplegar y operar el producto se amortiza rápido.

Donde PAM estorba es en los casos opuestos: una startup de ocho personas que despliega en una única cuenta cloud, con todo el mundo accediendo con su cuenta federada y MFA, no necesita montar un proxy de sesiones con flujo de aprobación en Slack. Ahí el rendimiento del control es negativo: añade fricción sin reducir riesgo real. Lo mismo ocurre con accesos puntuales de consultoría externa de dos semanas, donde un acceso federado con expiración automática es más barato que instrumentar un workflow PAM completo.

El antipatrón más común es el opuesto: adoptar PAM como checkbox de cumplimiento, documentar políticas rigurosas sobre el papel y luego crear "excepciones permanentes" para todos los administradores reales, dejando el control solo para los que no lo necesitan. Si al final de la implementación el inventario de excepciones es mayor que el de usuarios controlados, el proyecto ha fracasado aunque el auditor firme.

## Artículos relacionados en esta serie

- [Zero Trust](/posts/Zero-Trust/)
- [Gestión de secretos](/posts/Secret-Management/)
- [SPIFFE y SPIRE](/posts/SPIFFE-SPIRE/)
- [MFA](/posts/MFA/)
- [Keycloak, Okta y los IdP modernos](/posts/Keycloak-Okta-IdP/)

## Referencias

- NIST SP 800-53 Rev. 5 (septiembre 2020). *Security and Privacy Controls for Information Systems and Organizations*.
- NIST SP 800-207 (agosto 2020). *Zero Trust Architecture*.
- HashiCorp (2024). *Boundary Documentation*.
- Gravitational/Teleport (2024). *Teleport Access Platform Architecture*.
- AWS (2024). *IAM Identity Center User Guide*.
