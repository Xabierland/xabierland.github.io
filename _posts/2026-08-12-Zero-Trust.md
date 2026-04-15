---
title: "Zero Trust: la filosofía que convirtió el perímetro en un mito"
author: Xabierland
description: >-
    Zero Trust no es un producto ni una caja que se instala, es un modelo arquitectónico que asume la inexistencia del perímetro confiable y reemplaza la confianza implícita de la red por verificación continua basada en identidad, contexto y política. Entender sus orígenes, sus tenets y cómo se materializa con las primitivas modernas es lo que separa una implementación real de una etiqueta comercial.
date: 2026-08-12 10:00
categories: [DevSecOps, Seguridad de Aplicaciones]
tags: [Zero Trust, BeyondCorp, NIST, Arquitectura, Seguridad, Microservicios]
---

Pocos términos han sufrido tanto abuso comercial como "Zero Trust". Cualquier producto de seguridad con vocabulario actualizado se presenta como "solución Zero Trust", y el resultado es una confusión que oculta lo que el modelo propone. Zero Trust no es un firewall de nueva generación ni un VPN reempaquetado; es una posición arquitectónica que cambia lo que se considera confiable por defecto y traslada la frontera de la decisión desde la red hasta cada solicitud concreta. Este artículo forma parte de la serie [La evolución de la seguridad en aplicaciones modernas](/posts/La-evolucion-de-la-seguridad-en-aplicaciones-modernas/).

## Por qué el perímetro murió

Durante tres décadas, la seguridad corporativa se organizó como un castillo. Dentro del perímetro se confiaba; fuera, no. Un usuario que hubiera cruzado la puerta (VPN, LAN, oficina) se beneficiaba de privilegios implícitos: resolver hostnames internos, alcanzar APIs sin autenticación adicional, moverse lateralmente entre sistemas que asumían que cualquier tráfico del mismo segmento era legítimo. El modelo se apoyaba en dos premisas: que la amenaza venía del exterior y que, autenticado el usuario, confiar en él era razonable hasta fin de sesión.

Las dos premisas se derrumbaron en la década de 2010. Operation Aurora (2009), Target (2013), Sony (2014) y OPM (2015) compartían un patrón: comprometer un endpoint, obtener credenciales, moverse lateralmente durante meses porque dentro del perímetro nadie verificaba nada. La nube eliminó el concepto mismo de red interna: cargas en IaaS compartida, APIs expuestas por diseño, empleados accediendo desde cualquier dispositivo y ubicación. El perímetro había dejado de existir.

John Kindervag formalizó el concepto en un informe de Forrester de 2010 con la frase "never trust, always verify". La confianza no se hereda de la red, se establece en cada transacción. Google BeyondCorp, iniciado tras Operation Aurora y publicado desde 2014, demostró que una empresa grande podía operar sin VPN corporativa, tratando cada solicitud como si viniera de Internet abierto. En agosto de 2020, NIST publicó SP 800-207 *Zero Trust Architecture*, convirtiendo el concepto en un marco contrastable con vocabulario preciso y siete tenets canónicos.

## El mecanismo arquitectónico

NIST describe Zero Trust como un conjunto de principios en torno a un componente lógico central: un motor de decisión que evalúa cada solicitud de acceso en base a política y contexto, y un conjunto de puntos de aplicación que ejecutan esa decisión antes de permitir que la solicitud alcance al recurso. Si esto suena familiar es porque es exactamente la arquitectura PDP/PEP que ya [aparece en autorización clásica](/posts/Autenticacion-y-Autorizacion/), elevada a principio de diseño del sistema entero.

### Los siete tenets de NIST

Los tenets de SP 800-207 son el esqueleto. Primero: todas las fuentes de datos y servicios se consideran recursos; no hay recursos "internos" privilegiados. Segundo: toda la comunicación se asegura sin importar la ubicación de red. Tercero: el acceso se concede por sesión, no de forma permanente. Cuarto: el acceso se determina por una política dinámica que incluye identidad, cliente, atributos de recurso y señales de entorno. Quinto: la empresa monitoriza y mide la integridad y postura de todos los activos. Sexto: autenticación y autorización son dinámicas y estrictas antes de conceder acceso. Séptimo: la empresa recopila información sobre el estado de sus activos y la usa para mejorar su postura.

Los tenets no imponen tecnologías; describen propiedades del sistema. Un despliegue con VPN tradicional y reglas de firewall no satisface ninguno de forma significativa. Uno con identidad fuerte, mTLS universal, motor de política centralizado y telemetría continua los satisface casi todos por construcción.

### PDP y PEP en la arquitectura ZTA

El **PDP (Policy Decision Point)** recibe sujeto, recurso, acción y contexto y devuelve una decisión. NIST lo subdivide en *Policy Engine* (evalúa la política) y *Policy Administrator* (establece y gestiona la sesión). El **PEP (Policy Enforcement Point)** intercepta la solicitud, consulta al PDP y aplica el veredicto.

Separar estos roles permite que la política viva en un único lugar, versionada e inspeccionable, y distribuir PEPs por toda la arquitectura (gateway de ingreso, sidecar en cada pod, proxy delante de cada base de datos) sin duplicar la lógica. Es la misma separación que XACML lleva proponiendo desde 2003 y que OPA ejecuta hoy.

### Las primitivas que materializan Zero Trust

Zero Trust no es una implementación concreta, es un marco que se materializa combinando piezas que la serie ya ha tratado por separado. La **identidad fuerte de usuario** llega por OIDC, SAML o certificados de cliente, siempre verificada, nunca asumida. La **identidad de carga de trabajo** la aportan [SPIFFE y SPIRE](/posts/SPIFFE-SPIRE/), que entregan a cada servicio un SVID criptográfico que identifica *qué* está haciendo la llamada, no solo desde dónde. El **canal confiable** se establece con [mTLS](/posts/mTLS/) entre todos los componentes, eliminando la posibilidad de que un atacante en la red intermedia modifique o lea tráfico. La **decisión de autorización** la toma un motor externo (OPA, Cedar, OpenFGA) alimentado con identidad, contexto y política escritas como código. Los **puntos de aplicación** son los [API gateways, service meshes y sidecars](/posts/API-Gateway-BFF-Service-Mesh-Sidecar/) que interceptan cada llamada. La **telemetría y los riesgos** los aportan sistemas de observabilidad y detección que alimentan al PDP con señales en tiempo real.

Un despliegue que combine estas piezas está ejecutando Zero Trust en el sentido técnico del término. Un despliegue que tenga solo un VPN actualizado y un SSO corporativo no lo está, por mucho que el proveedor así lo etiquete.

### Micro-segmentación, ZTNA y verificación continua

La **micro-segmentación** aplica mínima exposición a nivel de red. La distinción entre L4 y L7 es importante: L4 (firewalls, *NetworkPolicy* de Kubernetes) permite o deniega por IP y puerto; una vez permitido el flujo, cualquier carga útil pasa. L7 (service mesh, gateways con reglas por ruta y método) decide según el contenido: cliente, método HTTP, ruta, cabeceras. Zero Trust a escala moderna requiere L7; L4 queda como defensa en profundidad.

**ZTNA (Zero Trust Network Access)** materializa el modelo para el acceso de usuarios a aplicaciones privadas. Frente a una VPN que coloca al usuario dentro de un segmento y le da acceso a todo lo alcanzable, ZTNA expone cada aplicación por separado a través de un broker que verifica identidad, postura y política por conexión. El usuario nunca está "dentro de la red". Una credencial VPN comprometida permite recorrer la red interna; una credencial ZTNA comprometida, solo las aplicaciones autorizadas para ese sujeto en ese momento. El radio de explosión se contrae drásticamente.

La dimensión temporal distingue Zero Trust de los modelos de sesión larga. El PDP puede reevaluar la sesión ante cambios en las señales: dispositivo fuera de cumplimiento, IP en país inesperado, comportamiento anómalo, credencial filtrada. La política puede exigir reautenticación, degradar privilegios o cortar la sesión. Esto requiere integración entre IdP, gestión de dispositivos, detección de amenazas y motor de política; pocas organizaciones lo tienen maduro, pero es lo que convierte Zero Trust en postura operativa real.

## Lo que Zero Trust no es

El término arrastra tantos malentendidos que conviene enunciar explícitamente lo que no es. **No es la ausencia de confianza**: es la eliminación de la confianza *implícita* derivada de la posición en la red, reemplazada por confianza *explícita* derivada de identidad verificada, contexto y política. **No es un producto**: no existe "un Zero Trust" que se compra e instala; es una arquitectura compuesta por identidad, política, cifrado y observabilidad. **No es VPN mejorado**: ZTNA reemplaza VPN para un caso de uso concreto, pero Zero Trust abarca mucho más (servicio a servicio, datos, contenedores, cargas de trabajo). **No es nuevo**: los principios existen desde hace décadas; lo nuevo es la conjunción de tecnologías que lo hacen operacionalmente viable a escala.

> Cuando un proveedor afirma que su producto "es Zero Trust", la primera pregunta útil es pedir el mapeo explícito de sus capacidades a los siete tenets de NIST SP 800-207. La respuesta distingue proveedores que entienden el modelo de los que usan la etiqueta.
{: .prompt-tip }

## Madurez y materialización en Kubernetes

CISA publicó un modelo de madurez Zero Trust con cinco pilares (identidad, dispositivos, redes, aplicaciones, datos) y cuatro niveles (tradicional, inicial, avanzado, óptimo). Identidad es lo más avanzado en la mayoría de organizaciones; datos lo más difícil, por requerir clasificación, etiquetado y control granular donde viven. Atravesar la matriz completa es trabajo de años y rara vez se completa a la vez en los cinco pilares.

Kubernetes ofrece terreno fértil para Zero Trust porque varias piezas encajan nativamente. SPIFFE/SPIRE asigna SVIDs a cada pod. Un service mesh (Istio, Linkerd, Cilium) impone mTLS por defecto en tráfico este-oeste y aplica autorización L7. Un *admission controller* con OPA Gatekeeper o Kyverno aplica política en el despliegue. Un gateway API verifica identidades de usuario en el borde. El resultado, bien ejecutado, es un clúster donde cada pod habla con cada pod solo si una política firmada lo permite, cada llamada lleva identidad criptográfica verificable y la red ha dejado de ser superficie de confianza. Es probablemente el ejemplo más claro hoy de Zero Trust materializado en producción.

## Cuándo es apropiado adoptarlo

Zero Trust es una dirección arquitectónica apropiada para prácticamente cualquier organización moderna, pero el ritmo y profundidad de adopción varían. Las señales que hacen urgente acelerar la adopción son claras: tráfico este-oeste entre servicios que hoy confía en la red subyacente, empleados remotos accediendo a aplicaciones corporativas por VPN, procesos regulatorios que exigen mínimos privilegios auditables, incidentes previos con movimiento lateral demostrado, migración a nube con recursos que coexisten con *on-premise*.

El escenario donde el coste supera el beneficio a corto plazo es el de sistemas pequeños, aislados y estables, con pocos usuarios y sin exposición externa significativa. En esos contextos, invertir en la pila completa de Zero Trust es prematuro; basta con los principios (identidad fuerte, cifrado, mínimo privilegio) sin la maquinaria asociada. La diferencia entre adoptar Zero Trust con sentido y hacerlo por moda se mide en si las decisiones de cada pieza se justifican frente a amenazas concretas o frente a la casilla del procurement.

## Cómo encaja con el resto de la arquitectura

Zero Trust es el marco que integra todas las primitivas que la serie ha tratado por separado. La autenticación (OIDC, SAML) aporta la identidad de usuario verificada que alimenta el PDP. La autorización (OPA, Casbin) es exactamente el PDP en acción. mTLS y SPIFFE proporcionan la identidad y el canal en la capa máquina a máquina. Los gateways, service meshes y sidecars son los PEPs distribuidos que aplican las decisiones. La admisión Kubernetes aplica política en el plano de control. La gestión de secretos y la identidad federada en la nube cierran el bucle sobre credenciales de larga duración que Zero Trust busca eliminar. Ninguna pieza por separado es Zero Trust; su integración sí lo es.

## Artículos relacionados en esta serie

- [mTLS](/posts/mTLS/)
- [SPIFFE y SPIRE](/posts/SPIFFE-SPIRE/)
- [API Gateway, BFF y Service Mesh](/posts/API-Gateway-BFF-Service-Mesh-Sidecar/)
- [Admission Control en Kubernetes](/posts/Admission-Control-Kubernetes/)
- [Autenticación y autorización](/posts/Autenticacion-y-Autorizacion/)

## Referencias

- Kindervag, J. (2010). *No More Chewy Centers: Introducing The Zero Trust Model Of Information Security*. Forrester.
- Ward, R. y Beyer, B. (2014). *BeyondCorp: A New Approach to Enterprise Security*. ;login:.
- NIST SP 800-207 (agosto 2020). *Zero Trust Architecture*.
- OMB (2022). *M-22-09 Federal Zero Trust Strategy*.
- CISA (abril 2023). *Zero Trust Maturity Model v2.0*.
- DoD. *Zero Trust Reference Architecture*.
