---
title: API Gateway, BFF, Service Mesh y Sidecar: patrones de seguridad en el perímetro y el interior
author: Xabierland
description: >-
    Los microservicios rompieron la idea del perímetro único. API Gateway, BFF, Service Mesh y Sidecar son cuatro patrones que reparten las responsabilidades de seguridad entre el norte-sur y el este-oeste. Entender qué resuelve cada uno, dónde se solapan y cómo se componen es lo que diferencia una arquitectura defendible de una colección de proxies.
date: 2026-06-17 10:00
categories: [DevSecOps, Seguridad de Aplicaciones]
tags: [API Gateway, BFF, Service Mesh, Sidecar, Proxy, Microservicios, Kubernetes]
---

Cuando una aplicación deja de ser un monolito y se descompone en decenas o cientos de servicios, la pregunta de dónde vive la seguridad deja de tener una respuesta obvia. El firewall perimetral sigue existiendo, pero dentro del clúster hay tráfico que nunca lo atraviesa. Este artículo forma parte de la serie [La evolución de la seguridad en aplicaciones modernas](/posts/La-evolucion-de-la-seguridad-en-aplicaciones-modernas/).

## Por qué el perímetro dejó de bastar

Durante décadas el modelo fue simple: un DMZ (Demilitarized Zone), un WAF (Web Application Firewall), un *load balancer* y detrás una red privada donde los servicios confiaban entre sí por el mero hecho de estar dentro. Ese modelo funcionaba porque el tráfico interno era previsible y el número de servicios cabía en la cabeza de un arquitecto.

La arquitectura de microservicios rompió las dos premisas. El tráfico interno pasó a representar órdenes de magnitud más volumen que el externo, y los servicios empezaron a desplegarse y retirarse a un ritmo incompatible con reglas de firewall estáticas. La distinción norte-sur (cliente a servicio) frente a este-oeste (servicio a servicio) se volvió operativa: son dos problemas con requisitos distintos de latencia, identidad y política. Forrester y NIST SP 800-207 (2020) formalizaron la conclusión: no se puede asumir confianza por ubicación de red. De ahí emergieron cuatro patrones complementarios.

## API Gateway: la puerta norte-sur

Un API Gateway es el punto único de entrada para el tráfico que viene de fuera. Su trabajo es consolidar las responsabilidades que antes se repetían en cada servicio: terminación TLS, validación de tokens OAuth 2.0 u OIDC, *rate limiting*, WAF, enrutamiento, transformación de protocolos, agregación de respuestas y observabilidad. Kong, Apigee, AWS API Gateway, Azure API Management o los basados en Envoy como Gloo Edge y Emissary son las implementaciones más extendidas.

La ventaja es clara: una aplicación que expone cien *endpoints* HTTP no necesita repetir cien veces la lógica de verificación de firmas JWT. El *gateway* la hace una vez, normaliza la identidad y reenvía al servicio una cabecera de confianza con los *claims* ya validados. Eso reduce superficie de ataque y centraliza la auditoría.

El riesgo, bien conocido por cualquiera que haya operado uno en producción, es que el *gateway* se convierta en un monolito de política. Cada equipo empuja sus reglas allí, los despliegues se serializan, los cambios de política bloquean entregas y el equipo de plataforma acaba siendo el cuello de botella del negocio. La mitigación pasa por separar plano de control y plano de datos, por exponer configuraciones declarativas versionadas, y por delegar en motores de política externos (OPA, Casbin) las decisiones de autorización que no pertenecen al *routing*.

## BFF: un gateway por tipo de cliente

El patrón *Backend For Frontend*, popularizado por Sam Newman alrededor de 2015 sobre prácticas de SoundCloud, es una variante del API Gateway con un propósito distinto. En lugar de un único *gateway* que sirva a todos los clientes, se despliega uno por tipo de cliente: uno para la aplicación móvil, otro para la web SPA (Single-Page Application), otro para integraciones de terceros. Cada BFF traduce las necesidades de su cliente a las capacidades del backend.

En el plano de seguridad, el BFF tiene un papel concreto y crítico: evitar que los *tokens* OAuth vivan en el navegador. El patrón BFF-cookie (descrito en el *OAuth 2.0 for Browser-Based Apps* draft del IETF, 2023) convierte al BFF en un cliente confidencial OAuth. El navegador solo maneja una cookie de sesión vinculada al BFF; el *access token* y el *refresh token* viven en el servidor. Esto neutraliza clases enteras de ataques de exfiltración de *tokens* desde XSS o extensiones comprometidas.

El mismo BFF almacena los *client secrets*, realiza el intercambio de código PKCE con el *Authorization Server*, gestiona el ciclo de vida del *refresh token* y realiza *token exchange* hacia servicios internos cuando la llamada requiere contexto de usuario. La consecuencia operativa es que el BFF se convierte en un componente de seguridad de primer orden, no un mero adaptador de forma.

## Service Mesh: el plano este-oeste

El *Service Mesh* resuelve el problema complementario al del *gateway*: qué pasa dentro del clúster cuando un servicio llama a otro. La idea es que cada *pod* tenga un *proxy sidecar* (típicamente Envoy) interceptando todo el tráfico entrante y saliente. Un plano de control (Istio, Linkerd, Consul Connect) distribuye configuración, identidad y políticas a esos *proxies*.

Lo que un *mesh* aporta en seguridad es, antes que nada, mTLS (mutual TLS) automático entre servicios. Cada *pod* recibe un certificado X.509 corto (a menudo rotado cada 24 horas o menos) con una identidad SPIFFE, y el *mesh* cifra y autentica todas las comunicaciones internas sin que los servicios escriban una línea de código TLS. Encima de eso se despliegan políticas L7: quién puede llamar a quién, con qué métodos HTTP, sobre qué rutas. Istio utiliza CRDs (Custom Resource Definitions) como `AuthorizationPolicy`; Linkerd apuesta por políticas de servidor más minimalistas.

El precio ha sido históricamente el *sidecar*: un contenedor Envoy por cada *pod*, con su consumo de memoria y CPU, su ciclo de vida y sus modos de fallo. La tendencia actual, encarnada por Istio Ambient Mesh (GA en 2024) y por Cilium Service Mesh basado en eBPF, es moverse hacia modelos sin *sidecar* donde el plano de datos vive en nodos o en el kernel. Esa evolución reduce la huella pero introduce sus propias complicaciones de aislamiento y diagnóstico.

> Un Service Mesh no sustituye al API Gateway. Resuelven problemas distintos y conviven: el gateway gobierna la entrada, el mesh gobierna el interior.
{: .prompt-info }

## El patrón Sidecar como mecanismo subyacente

El *sidecar* no es solo una pieza del *mesh*; es un patrón general que consiste en desplegar un proceso auxiliar junto al proceso principal, compartiendo ciclo de vida y, en Kubernetes, el mismo *pod*. Más allá del *proxy* de red, hay *sidecars* para inyección de secretos (Vault Agent), para agregación de *logs*, para autenticación específica de protocolo (como *oauth2-proxy*) o para terminación mTLS en clientes que no la soportan nativamente.

Las ventajas del patrón son dos: transparencia para el proceso principal (no tiene que cambiar) y reutilización (un mismo *sidecar* se usa para decenas de servicios). Las desventajas se acumulan a escala: cada *sidecar* multiplica el consumo de recursos por el número de *pods*, añade latencia a cada llamada, complica el diagnóstico cuando algo falla entre contenedores, y obliga a sincronizar inicio y terminación para evitar tráfico durante *bootstrap*. Kubernetes 1.29 (2023) introdujo los *native sidecar containers* precisamente para resolver el problema de secuencia de arranque y apagado que había atormentado a los operadores durante años.

## Proxy, el primitivo común

Debajo de los tres patrones anteriores hay un mismo concepto: un *proxy*. Un *reverse proxy* (Nginx, HAProxy, Envoy, Traefik) se sitúa delante de un servicio y expone su funcionalidad; un *forward proxy* (Squid, o los egress gateways del mesh) se sitúa delante de un cliente y controla su salida. El API Gateway es un *reverse proxy* especializado; el *sidecar* del mesh actúa como ambas cosas en la misma instancia. Entender que los cuatro patrones son composiciones de *proxies* con distintos ámbitos ayuda a razonar sobre dónde colocar cada responsabilidad.

## Cuándo aplicar cada patrón

No toda arquitectura necesita los cuatro. Una aplicación con un puñado de servicios y un único frontal web puede vivir perfectamente con un API Gateway bien configurado y nada más. Un sistema con múltiples clientes nativos y una SPA suma un BFF al cuadro. Cuando los servicios internos superan la docena, cuando cruzan zonas de confianza (varios *namespaces*, varios clústeres) o cuando la regulación exige cifrado extremo a extremo, aparece el caso para *service mesh*.

### Matriz de decisión razonable

Como orientación: menos de diez servicios, un cliente y un equipo, basta con *reverse proxy* o *gateway* ligero. Diez a cincuenta servicios, varios clientes, dos o tres equipos, justifica gateway dedicado más BFFs. Más de cincuenta servicios, varios clústeres o requisitos de mTLS extremo a extremo, justifica mesh. El coste operativo de un *mesh* (curva de aprendizaje, SRE especializado, actualizaciones continuas) no se amortiza por debajo de ese umbral.

### Reparto de responsabilidades

La regla que mejor funciona en producción es asignar cada responsabilidad al punto más cercano al problema. La autenticación del usuario ocurre en el *gateway* o BFF, donde llega el *token* externo. La autorización fina, cuando depende del recurso, vive en el servicio o en un PDP externo consultado por este. El *rate limiting* de APIs públicas vive en el *gateway*; las cuotas internas entre servicios, en el *mesh*. El mTLS entre servicios es del *mesh*; la terminación TLS del tráfico externo, del *gateway*. La observabilidad se reparte: *traces* distribuidos cruzan todos los planos y obligan a propagar cabeceras W3C Trace Context consistentemente.

> Cuando el mismo control de acceso se escribe en el gateway y se repite en el servicio, uno de los dos está de más o ambos divergirán con el tiempo.
{: .prompt-warning }

## Relación con identidad de carga de trabajo

El *mesh* no inventa la identidad de los servicios; la consume. SPIFFE (Secure Production Identity Framework For Everyone) y su implementación SPIRE son el estándar que proporciona a cada carga de trabajo un identificador verificable (un SVID, típicamente un certificado X.509 o un JWT). Istio, Linkerd y Consul integran SPIFFE bien por defecto o como capa opcional. Esa separación entre emisor de identidad y consumidor es lo que permite que un servicio pueda ser identificado de forma coherente por el *mesh*, por un *vault* de secretos y por un *gateway* externo sin duplicar lógica.

## Consideraciones de rendimiento

Cada salto añade latencia. Un par típico de *sidecars* (ingreso y egreso del mismo *pod*) introduce del orden de cientos de microsegundos a un milisegundo en cada llamada, dependiendo de si hay TLS y de las políticas L7 aplicadas. Para aplicaciones con miles de llamadas por petición de usuario, el coste se acumula. Los modelos sin *sidecar* o basados en eBPF reducen ese coste en parte, pero no lo eliminan. La pregunta que hay que hacerse no es si el *mesh* tiene coste, sino si el coste es aceptable dado lo que compra a cambio.

## Artículos relacionados en esta serie

- [mTLS](/posts/mTLS/)
- [SPIFFE y SPIRE](/posts/SPIFFE-SPIRE/)
- [CASBIN, OPA y OpenFGA](/posts/CASBIN-OPA-OpenFGA/)
- [Zero Trust](/posts/Zero-Trust/)
- [OAuth 2.0](/posts/OAuth2/)

## Referencias

- NIST SP 800-207 (2020). *Zero Trust Architecture*.
- IETF (2023). *draft-ietf-oauth-browser-based-apps: OAuth 2.0 for Browser-Based Apps*.
- Istio (2024). *Documentation: Security and Ambient Mesh*.
- Linkerd. *Documentation: Policy and mTLS*.
- Kubernetes (2023). *KEP-753: Sidecar Containers (GA 1.29)*.
- Newman, S. (2015). *Pattern: Backends For Frontends*.
