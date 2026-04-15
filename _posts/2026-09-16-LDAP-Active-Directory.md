---
title: "LDAP y Active Directory: el directorio empresarial que precede (y sobrevive) a los IdPs modernos"
author: Xabierland
description: >-
    LDAP (Lightweight Directory Access Protocol) es el estándar que modela la identidad corporativa como un árbol jerárquico consultable. Resuelve el problema de centralizar usuarios, grupos y recursos en un único servicio replicable, con décadas de despliegue en banca, administración pública e industria. Aunque los IdPs modernos hablan OIDC o SAML hacia las aplicaciones, LDAP sigue siendo el repositorio maestro subyacente en una parte abrumadora del tejido corporativo.
date: 2026-09-16 10:00
categories: [DevSecOps, Seguridad de Aplicaciones]
tags: [LDAP, Active Directory, Directorio, Identidad, Kerberos, Entra ID, IAM]
---

Este artículo forma parte de la serie [La evolución de la seguridad en aplicaciones modernas](/posts/La-evolucion-de-la-seguridad-en-aplicaciones-modernas/). Hablar de identidad empresarial sin hablar de LDAP es reescribir la historia. Antes de que SAML existiera, antes de que OIDC se normalizase, antes incluso de que la web fuera el canal dominante, las organizaciones ya tenían directorios: bases de datos jerárquicas que describían personas, equipos, impresoras, salas y grupos. LDAP nació como el protocolo estándar para consultarlas, y Active Directory lo convirtió en infraestructura ubicua en el mundo Windows. Treinta años después siguen ahí, bajo los IdPs modernos, alimentándolos.

## Por qué existe el concepto de directorio

A finales de los ochenta la ITU definió X.500, una visión enciclopédica de un directorio global interoperable accedido mediante DAP (Directory Access Protocol). DAP era exhaustivo pero pesado: requería pila OSI completa y capacidades que ningún cliente ligero de la época podía permitirse. En 1993 apareció en la Universidad de Michigan una alternativa más digerible pensada para correr sobre TCP/IP: Lightweight Directory Access Protocol, LDAP. La versión 3, estandarizada en RFC 2251 en 1997 y reformulada en RFC 4511 (junio 2006), es la base de todo lo que se usa hoy.

La motivación práctica era clara. Cada aplicación corporativa mantenía su propio almacén de usuarios, con consecuencias conocidas: identidades duplicadas, contraseñas divergentes, bajas que no se propagaban, auditorías imposibles. Un directorio centralizado convertía la identidad en un servicio compartido: una persona entraba una vez en nómina, aparecía en el directorio, y todas las aplicaciones podían consultarlo como fuente única. El mismo razonamiento aplicaba a grupos (para autorización gruesa) y a recursos (impresoras, buzones, equipos).

## El modelo de información

Un directorio LDAP es un árbol, el DIT (Directory Information Tree). Cada nodo es una entry con un nombre distintivo, el DN (Distinguished Name), compuesto por RDNs (Relative Distinguished Names) concatenados desde la hoja hasta la raíz. Una entry típica podría identificarse con un DN como el que describe a un empleado dentro de una unidad organizativa, dentro de un dominio. Los componentes habituales son cn (common name), ou (organizational unit), o (organization) y dc (domain component).

Cada entry tiene atributos tipados. Los atributos se definen en schemas, que a su vez se componen de object classes. Una entry puede ser inetOrgPerson, posixAccount, user (en AD), group, organizationalUnit, entre muchas otras. Las object classes determinan qué atributos son obligatorios y cuáles opcionales. Un usuario tiene típicamente uid, sAMAccountName o userPrincipalName, mail, memberOf, y decenas de campos más según el schema.

El protocolo es binario (codificación BER sobre TCP, puerto 389 en claro o 636 sobre TLS, más el modo StartTLS sobre 389). Las operaciones centrales son bind (autenticación), search (consultas con filtros como los descritos en RFC 4515), compare, add, modify, delete, modify DN y unbind. Los filtros LDAP son muy expresivos y se pueden anidar con operadores lógicos, lo que los convierte tanto en una herramienta potente como en una fuente clásica de vulnerabilidades por inyección cuando se construyen concatenando strings sin escapar.

## Bind modes y autenticación

El bind es el mecanismo por el que un cliente prueba quién dice ser. Hay tres grandes modos. El simple bind envía DN y contraseña en texto, dependiendo por completo de TLS para no ser un regalo al primer sniffer de la red. El anonymous bind permite operaciones de lectura sin credenciales, útil para directorios públicos y peligroso si se deja activado sin pensarlo. El SASL bind (Simple Authentication and Security Layer) abre la puerta a mecanismos robustos: GSSAPI con Kerberos, DIGEST-MD5, EXTERNAL con certificados cliente tras mTLS, o PLAIN dentro de un canal ya cifrado.

> En entornos corporativos la combinación habitual es SASL GSSAPI sobre Kerberos, precisamente para que el cliente nunca envíe la contraseña al directorio. El directorio confía en el ticket emitido por el KDC, que a su vez deriva su confianza de la clave compartida con cada principal.
{: .prompt-info }

## Active Directory y su huella

Microsoft lanzó Active Directory con Windows 2000 y redefinió lo que una organización esperaba de un directorio. No solo ofrecía LDAP como protocolo de consulta, sino que integraba autenticación Kerberos, políticas de grupo (GPO), DNS dinámico, replicación multimaster y un modelo de forest con trusts entre dominios. Una organización típica gestiona miles de usuarios, decenas de miles de equipos y un volumen masivo de permisos aplicados a recursos Windows mediante ese mismo directorio. Las entries de AD se exponen por LDAP con atributos como sAMAccountName, userPrincipalName, objectSid, memberOf, pwdLastSet, userAccountControl y muchos otros específicos del schema de Microsoft.

La replicación en AD es multi-master entre controladores de dominio (DC), con roles FSMO para operaciones que requieren un único árbitro. Los DCs se ubican por sitios AD, correspondientes a topologías de red, para optimizar el tráfico de replicación y de autenticación. El resultado es una arquitectura resistente y rápida dentro de la intranet corporativa, donde un cliente puede localizar el DC más cercano vía DNS SRV y autenticarse contra él en milisegundos.

Otras implementaciones de directorio conviven con AD. OpenLDAP es el referente open source para Unix, con su configuración basada en slapd y su backend mdb. 389 Directory Server (antes Fedora DS, derivado de Netscape Directory) sigue activo en el ecosistema Red Hat. Apache Directory Server es la alternativa en Java. En el mundo Linux es habitual integrar autenticación de sistema con SSSD o nslcd contra cualquiera de estos directorios.

## Dónde queda LDAP hoy

La historia habitual es la siguiente. La organización tenía Active Directory desde los 2000. Con la llegada de SaaS, se desplegó un IdP moderno (ver [Keycloak, Okta y los IdP modernos](/posts/Keycloak-Okta-IdP/)) para hablar SAML y OIDC hacia las aplicaciones. El IdP no reinventó el almacén de usuarios: se conectó al AD existente mediante LDAP (o mediante un agente de sincronización). A partir de ahí, cuando un usuario hace login en una SaaS, su autenticación puede acabar resolviéndose contra AD bajo las capas, pero el protocolo que ven las aplicaciones es OIDC.

Microsoft extendió esta línea con Entra ID (antes Azure Active Directory), que no es LDAP ni habla LDAP en su superficie nativa. Entra ID es un directorio cloud con APIs REST, SCIM para aprovisionamiento y protocolos modernos para aplicaciones (OIDC, OAuth 2.0, SAML). Para aplicaciones legadas que todavía requieren LDAP, ofrece Entra Domain Services, una envoltura que emula LDAP y Kerberos sobre el directorio cloud. La dirección estratégica es clara, pero la realidad operativa sigue incluyendo LDAP.

> LDAP no está muerto. Está enterrado bajo capas más modernas, y esa es la razón por la que los incidentes que lo afectan (credenciales de servicio filtradas, binds con cuentas privilegiadas, inyección LDAP en formularios internos) siguen apareciendo en informes anuales de brechas.
{: .prompt-warning }

## Cuándo usar LDAP directamente y cuándo no

Tiene sentido exponer LDAP como protocolo cuando se integra con software legado que no habla OIDC ni SAML y que no se puede modificar, cuando se opera infraestructura Unix que depende de PAM/NSS contra directorio, o cuando se construyen servicios internos en la intranet donde el cliente es confiable y el directorio es la fuente autoritativa. En estos casos, TLS obligatorio, SASL en lugar de simple bind y cuentas de servicio con permisos mínimos son mínimos no negociables.

No tiene sentido usar LDAP directamente como autenticación para aplicaciones web modernas, ni exponerlo a internet, ni integrarlo con SaaS. Para ese propósito existe el IdP. Tampoco tiene sentido implementar autorización fina leyendo atributos LDAP dentro de cada aplicación: el directorio modela pertenencia gruesa (grupos, unidades), y la autorización fina pertenece a otras capas (ver [RBAC, ABAC y ReBAC](/posts/RBAC-ABAC-ReBAC/)).

Hay un anti-patrón habitual: aplicaciones que hacen simple bind con las credenciales del usuario final contra el directorio corporativo como forma de login. Funciona, pero traslada la contraseña del usuario a la aplicación en cada login, multiplica la superficie de exposición y dificulta MFA. Si el directorio está bajo un IdP, la aplicación debería hablar con el IdP, no con el directorio.

## Encaje con el resto de la arquitectura

En una arquitectura actual, el directorio (sea AD, OpenLDAP, Entra ID) es el sistema de registro de identidades humanas. El IdP se conecta a él para poblar su base de usuarios y delegar bind en autenticaciones interactivas, añadiendo encima MFA, políticas de acceso y emisión de tokens. Las aplicaciones hablan con el IdP mediante [OIDC](/posts/OIDC/) o [SAML](/posts/SAML/). [Kerberos](/posts/Kerberos/) sigue presente en el flanco corporativo cuando hay AD. El aprovisionamiento hacia SaaS se hace con [SCIM](/posts/SCIM/), que lee del directorio y empuja a las aplicaciones.

Esa estratificación tiene una consecuencia útil: el directorio deja de ser un cuello de botella para cada aplicación y se convierte en una pieza específica con un rol claro. Y cuando en los próximos años se hable de migraciones a directorios cloud, de federación cruzada entre tenants de Entra ID, o de arquitecturas híbridas con AD Connect, la conversación seguirá girando en torno a los mismos objetos que LDAP nombró en 1993.

## Artículos relacionados en esta serie

- [Kerberos](/posts/Kerberos/)
- [Keycloak, Okta y los IdP modernos](/posts/Keycloak-Okta-IdP/)
- [SAML](/posts/SAML/)
- [SCIM](/posts/SCIM/)

## Referencias

- RFC 4511 (junio 2006). *Lightweight Directory Access Protocol (LDAP): The Protocol*.
- RFC 4515 (junio 2006). *Lightweight Directory Access Protocol (LDAP): String Representation of Search Filters*.
- RFC 4513 (junio 2006). *Lightweight Directory Access Protocol (LDAP): Authentication Methods and Security Mechanisms*.
- ITU-T (1988, revisiones sucesivas). *Recommendation X.500: The Directory*.
- Microsoft (documentación vigente). *Active Directory Domain Services Technical Specification*.
