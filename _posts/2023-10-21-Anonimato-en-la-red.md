---
title: Anonimato en la red
description: >-
  Guía para conseguir el anonimato en la red.
author: Xabierland
date: 2023-10-21
categories: [Blog]
tags: [Anonimato, Privacidad, Seguridad]
---

## Introduccion

La seguridad en internet es algo que ha ido desapareciendo con el paso del tiempo. Atrás queda el tiempo en que la red era un sitio anónimo y descentralizado, en el que todo era por personas y para personas, dando paso a una red prácticamente centralizada y vigilada por agencias gubernamentales o grandes compañías de tecnología. Este post intentará volver atrás en el tiempo en algunos aspectos para así poder avanzar hacia un futuro más seguro, privado y anónimo.

### Objetivo

El objetivo de este post es crear un punto de referencia para todas aquellas personas que quieran empezar a usar la red de forma anónima y privada. Esta guía no es para todo el mundo; si no eres un usuario medianamente avanzado en informática y no tienes conocimientos de redes, criptografía, sistemas operativos, etc., esta guía no es para ti.

Esta guía puede resultarte útil si te dedicas al periodismo y necesitas cierta anonimidad para tus investigaciones, si perteneces a alguna organización activista y buscas reivindicar tus derechos, si eres un whistleblower y quieres denunciar algún tipo de corrupción, o simplemente si eres un usuario con conocimientos y conciencia que desea recuperar su privacidad en la red.

### Disclaimer

Antes de empezar con la guía, quiero dejar claro que mis conocimientos sobre la materia no son absolutos. Todo lo que voy a explicar en este post es lo que he aprendido con el tiempo y la experiencia. Si alguien ve algún error o tiene alguna sugerencia, no dude en contactar conmigo.

## Hardware

Aunque el hardware sea lo primero que vamos a tratar en este post, es la parte menos importante y a la vez la más difícil de conseguir de forma anónima. Mi recomendación, en caso de que estés muy interesado en obtener el hardware de forma anónima, es:

- Comprar el hardware en una tienda física y pagar en efectivo. No es la mejor opción, ya que el dinero fiat es rastreable, pero es la forma más fácil de conseguirlo.
- Comprar el hardware en una tienda usando una tarjeta de regalo anónima. Ten en cuenta que no puedes dar datos personales al comprar de esta manera, ya que perdería su propósito.
- Comprar el hardware en línea mediante criptomonedas. Esta es la forma más difícil, ya que no hay muchas tiendas que acepten criptomonedas y las que lo hacen suelen pedir datos personales.
- Comprar el hardware en línea de segunda mano. Depende mucho de la persona que lo venda, y es complicado encontrar a alguien que no pida datos personales, acepte criptomonedas, envíe de forma anónima y, además, no sea un estafador.

Una vez vistos los métodos de adquisición que conozco, pasemos a analizar qué hardware es el más adecuado para conseguir el anonimato en la red.

### Ordenador de sobremesa

El ordenador de sobremesa es quizá el más fácil de conseguir, ya que puedes ir comprando las piezas poco a poco en diferentes tiendas y montarlo tú mismo en casa. También tiene la ventaja de que suelen ser, en general, más potentes que un ordenador portátil, por lo tanto, son más útiles para tareas que requieren mayor potencia. Además, son fáciles de modificar y ampliar.

Como desventaja, tenemos que es un ordenador que no es fácil de transportar y ocupa bastante espacio. Al estar en un mismo lugar, es más fácil que pueda ser localizado. También es más difícil de ocultar en caso de que sea necesario.

### Ordenador portatil

El ordenador portátil es quizá el más difícil de conseguir de forma anónima, ya que, por lo general, se compra en una tienda física o en línea y suele pagarse con tarjeta de crédito o débito. También es posible comprarlo en efectivo, pero es más difícil encontrar una tienda que acepte efectivo que una que acepte tarjeta.

Es recomendable elegir un ordenador sin sistema operativo, ya que así evitamos tener que formatear el disco duro y nos aseguramos de que no haya spyware preinstalado.

Entre sus ventajas está su gran movilidad y adaptabilidad. Es fácil de transportar y ocultar en caso de que sea necesario.

Por otro lado, sacrificamos la potencia y la capacidad de modificación.

### Dispositivo Movil

Dentro de los dispositivos móviles englobamos tanto teléfonos como tabletas. Estos, por lo general, suelen venir con Android, por lo que es recomendable cambiar el sistema operativo por uno más seguro y privado, como GrapheneOS. También es posible encontrar móviles ya fabricados con otros forks de Android con capas de seguridad, aunque, al estar fabricados por empresas privadas, no se puede garantizar que no tengan spyware.

- [GrapheneOS](https://grapheneos.org/)

El dispositivo móvil es muy importante, ya que es el dispositivo que más usamos en el día a día y siempre llevamos con nosotros. Si no tenemos cuidado, puede ser un gran punto de fuga de información.

## Software

Cuando hablamos de software anónimo y privado, es imposible no hablar de software libre y de software abierto. Aunque no son lo mismo, ambos comparten la filosofía de libertad y privacidad. Es por ello que todos los programas que voy a recomendar en este post son software libre y/o software abierto.

### OS

Un sistema operativo es un conjunto de programas que permiten la comunicación entre el usuario y el hardware.

Elegir un sistema operativo es muy complicado: hay muchas opciones y cada una tiene sus ventajas y desventajas. Además, hay mucho "fandom" en torno a los sistemas operativos y es difícil encontrar una opinión objetiva sobre ellos, esta incluida.

Para elegir un sistema operativo, intentaría buscar uno que sea libre y que no tenga detrás una empresa con intereses en recopilar información sobre ti. También es importante que el sistema operativo sea lo más transparente posible y que no tenga programas preinstalados que puedan ser usados para recopilar información.

Como no quiero entrar en debates y no quiero que este post se convierta en una discusión sobre cuál sistema operativo es mejor o peor, voy a dejar una lista de sistemas operativos que cumplen con los requisitos anteriores y que son buenas opciones a la hora de elegir un sistema operativo:

- [Debian](https://www.debian.org/)
- [Arch Linux](https://www.archlinux.org/)
- [Gentoo](https://www.gentoo.org/)
- [Linux From Scratch](https://www.linuxfromscratch.org/)
- [FreeBSD](https://www.freebsd.org/)
- [OpenBSD](https://www.openbsd.org/)
- [Tails](https://tails.net/)
- [Whonix](https://www.whonix.org/)
- [Qubes OS](https://www.qubes-os.org/)

### VPS

Un VPS es un servidor virtual privado. Es una máquina virtual que se encuentra en un servidor remoto y que puedes usar para lo que quieras.

A la hora de elegir un proveedor de VPS, es importante que elijas uno que no pida datos personales al registrarte. También es recomendable que permitan el pago con criptomonedas, preferiblemente Monero, y que permita el acceso mediante Tor. Además, es importante que el proveedor no esté ubicado en un país que no proteja la privacidad de sus usuarios. Suiza, Estonia e Islandia son buenas opciones.

A continuación te dejo una lista de proveedores de VPS que cumplen con los requisitos anteriores:

| Servicio | Tor | Cripto | País |
| --- | --- | --- | --- |
| Privex | X | X | 
| Njalla | X | X |
| IncogNet | X | X |
| 1984 | | X |

### VPN

Si estás leyendo este "post" mientras estás conectado a una VPN como NordVPN, ExpressVPN, CyberGhostVPN... déjame que salude a la empresa e incluso a la agencia de inteligencia que tienes detrás.
Antes de seguir leyendo, hazte un favor y desconéctate de tu VPN para que pueda contarte cómo hacer las cosas bien.

La forma correcta de montar y utilizar una VPN para mantenerte anonimo es mediante un VPS. De esta forma, tú eres el dueño del servidor y no tienes que confiar en nadie más. Si además, como hemos visto en el punto anterior, usamos una VPS adquirida de forma anónima, tenemos una VPN mucho más segura y privada que cualquier VPN comercial.

Te dejo enlaces de dos tecnologías para ayudarte a configurar tu VPN:

- [OpenVPN](https://openvpn.net/)
- [WireGuard](https://www.wireguard.com/)

Recuerda configurar tu VPN para que no guarde logs y para que no tenga fugas de DNS. También es importante que la VPN y por tanto tu VPS esté ubicada en un país que proteja la privacidad de sus usuarios como he mencionado en el punto anterior.

Si aún así deseas usar una VPN comercial, la única opción que puedo recomendar es Mullvad. Es una VPN que no pide datos personales al registrarte, acepta Monero como método de pago, tiene politicas de privacidad muy estrictas, no guarda logs y ofrece tuneles con cifrado cuantico.

- [Mullvad](https://mullvad.net/)

> [!NOTE]
> Recuerda que cuanto más raros sean tus metodos de conexion más sobresaldras en la red. La clave y el punto mas complicado de todo esto es encontrar el equilibrio entre la seguridad y la privacidad sin llamar la atención.

### Proxy

Un proxy es un servidor que actúa como intermediario entre el cliente y otros servidores. Es una forma de ocultar tu IP y de acceder a contenido bloqueado en tu país.

Las redes de proxies más conocidas son Tor y I2P. Ambas son redes descentralizadas y anónimas que permiten a los usuarios acceder a la red de forma anónima y privada.

- [Tor](https://www.torproject.org/)
- [I2P](https://geti2p.net/)

También quiero aprovechar para dar respuesta a la pregunta de ¿Es recomendable usar Tor con una VPN? La respuesta es no. La única razón por la que querrías hacer esto es si vives en un país donde el uso de Tor no solo está prohibido, sino que está bloqueado, como China o Irán. En este caso, podría recomendarse usar una VPN, aunque Tor ofrece Bridges, que es una forma de acceder a la red Tor en países en los que está bloqueada. China ha demostrado ser capaz de bloquear también los Bridges. En cualquier otro caso, conectar una VPN a Tor solo levantaría sospechas.

Otra opcion es usar Snowflake. Snowflake es un proyecto de Tor que permite a los usuarios de Tor compartir su conexión con otros usuarios de Tor que estén en países donde Tor está bloqueado. De esta forma, se puede acceder a la red Tor en estos países.

- [BridgeDB](https://bridges.torproject.org/)
- [Snowflake](https://snowflake.torproject.org/)

### Navegador

Los navegadores son una herramienta crucial que usamos a todas horas, por lo que elegir uno no es moco de pavo. A continuación, te dejo una lista de navegadores a considerar al elegir el tuyo.

| Navegador          | Licencia | UI  |
|--------------------|----------|-----|
| Mozilla Firefox    | MPL      | GUI |
| Ungoogled Chromium | BSD      | GUI |
| GNU IceCat         | GPL      | GUI |
| LibreWolf          | MPL      | GUI |
| BadWolf            | BSD      | GUI |
| Mullvad            | BSD      | GUI |
| Tor Browser        | BSD      | GUI |
| w3m                | MIT      | TUI |
| Links              | GPL      | TUI |
| Lynx               | GPL      | TUI |

Firefox es el navegador más conocido de la lista, pero hay que tener en cuenta que no está enfocado en la privacidad ni el anonimato. Es posible configurarlo e instalarle extensiones para "reforzarlo", pero aun así no es la mejor de las opciones.

UnGoogled Chromium es un fork de Chromium que elimina todo el spyware de Google y añade algunas funcionalidades de privacidad. Es una buena opción si no quieres complicarte mucho la vida aunque para eso prefiero cualquiera de los siguientes navegadores.

GNU IceCat es una versión completamente libre del navegador web Mozilla Firefox, promovida y mantenida por el Proyecto GNU. Al ser una alternativa derivada de Firefox, IceCat incluye funcionalidades y características que buscan reforzar el respeto a la privacidad y la libertad del usuario, eliminando todo el software y complementos que no cumplen con los estándares de software libre establecidos por la Free Software Foundation. IceCat incorpora de serie GNU LibreJS, una extensión que bloquea scripts JavaScript que no son libres. Esto es especialmente útil para usuarios que desean asegurarse de que los scripts en las páginas que visitan cumplen con los estándares de software libre.

LibreWolf es otro navegador web de código abierto y una bifurcación de Mozilla Firefox, pero a diferencia de GNU IceCat, LibreWolf se centra principalmente en ofrecer una experiencia de navegación que maximice la privacidad y la seguridad del usuario, sin un enfoque tan estricto en cumplir exclusivamente con software libre. LibreWolf es desarrollado y mantenido por una comunidad de voluntarios y es popular entre aquellos que buscan mejorar su privacidad en línea sin alejarse demasiado del ecosistema de Firefox.

BadWolf es un navegador web minimalista y de código abierto, diseñado con un enfoque diferente al de navegadores convencionales como Firefox o Chrome. A diferencia de estos navegadores grandes y repletos de funciones, BadWolf apuesta por la simplicidad, la eficiencia y un diseño limpio que prioriza la seguridad y la privacidad sin recurrir a una gran cantidad de opciones y personalizaciones. BadWolf es un proyecto más reciente y menos conocido que navegadores como LibreWolf o GNU IceCat, y se orienta hacia usuarios que buscan un navegador ligero y rápido que no dependa de bibliotecas y funcionalidades web avanzadas.

El Tor Browser es un navegador de código abierto desarrollado por el Proyecto Tor. Es una versión modificada de Mozilla Firefox que se conecta automáticamente a la red Tor (The Onion Router) para proteger la privacidad y el anonimato del usuario en línea.

El Mullvad Browser es un navegador desarrollado en colaboración con el Proyecto Tor y la empresa de VPN Mullvad. Se basa en el mismo código y enfoque de privacidad que Tor Browser, pero no utiliza la red Tor. En su lugar, está diseñado para usarse en combinación con la red de servidores VPN de Mullvad (aunque puede usarse con otras VPN o incluso sin VPN), brindando una capa de anonimato diferente.

w3m, Links y Lynx son navegadores de texto que no soportan JavaScript ni CSS. Son muy útiles para navegar por la web de forma anónima y privada, ya que no permiten que los sitios rastreen al usuario. También son muy útiles para navegar por la web en dispositivos con poca potencia o sin interfaz gráfica.

También es recomendable que al usar cualquier navegador de la lista deshabilitemos JavaScript y las cookies, ya que ambos son métodos muy usados para rastrear a los usuarios y obtener información sobre ellos. Los más celosos de su seguridad en la web buscan, en parte, hacer una vuelta a los orígenes de la web con la Web 1.0 y plantean el uso de XHTML y CSS para hacer nuestras páginas web. Richard Stallman es un gran defensor de esta idea y crítico del uso de JavaScript en la web. [La trampa de JavaScript](https://www.gnu.org/philosophy/javascript-trap.html){:target="_blank"}

#### Extensiones

Si decides usar navegadores menos enfocados en la privacidad y el anonimato, es recomendable que instales extensiones que te ayuden a conseguirlo. A continuación, te dejo una lista de extensiones a considerar.

- uBlock Origin
- Privacy Badger
- User-Agent Switcher
- NoScript
- HTTPS Everywhere
- Canvas Blocker
- xBrowserSync
- LibreWolf Updater
- Chaff

### Correo

Servicios de correo como Gmail, Outlook, Yahoo, etc., son servicios que no deberíamos usar nunca. Estos servicios están obligados por ley a dar información sobre sus usuarios a las agencias de inteligencia, además de que no son software libre y, por lo tanto, no se puede garantizar que no tengan spyware.

Es difícil dar una recomendación realmente buena sobre cómo usar el correo de forma anónima y privada. Las cuatro opciones que se me ocurren son usar un servidor de correo propio, ProtonMail, Tutanota o OnionMail.

- [ProtonMail](https://protonmail.com/)
- [Tutanota](https://tutanota.com/)
- [OnionMail](https://onionmail.info/paper.html)

### Mensajeria

Si quieres privacidad y anonimato, olvídate de WhatsApp, Telegram, Signal, Discord, Slack... Todas estas aplicaciones están obligadas por ley a dar información sobre sus usuarios a las agencias de inteligencia, además de que no son software libre y, por lo tanto, no se puede garantizar que no tengan spyware.

La mejor opción es usar IRC. IRC es un protocolo de comunicación que permite la comunicación entre usuarios mediante texto plano. Es un protocolo muy antiguo que no ha cambiado mucho desde su creación. Es un protocolo simple que no tiene muchas funcionalidades, pero es muy seguro y privado. Además, es software libre y hay muchos clientes de IRC disponibles para todos los sistemas operativos.

Otras opciones son XMPP y Matrix. Ambos son protocolos de comunicación que permiten la comunicación entre usuarios mediante texto plano. Son protocolos más modernos que IRC y tienen más funcionalidades, pero también son más complejos y, por lo tanto, más difíciles de usar. Aun así, IRC sigue siendo la mejor opción en cuanto a privacidad y anonimato.

### Criptomonedas

Aunque en general el concepto de criptomoneda es bastante seguro y privado, hay ciertas criptomonedas que son más privadas que otras. Yo, personalmente, recomiendo Monero. Es una criptomoneda que ofrece un gran anonimato y privacidad. Además, está en constante desarrollo y tiene una gran comunidad detrás. También es una buena opción Zcash.

- [Monero](https://www.getmonero.org/)
- [Zcash](https://z.cash/)

### Gestor de archivos

Aunque no es una herramienta que se suela asociar con la privacidad y el anonimato, es recomendable usar un gestor de archivos que sea software libre.

| Gestor de archivos | Licencia |
|--------------------|----------|
| Ranger | GPL |
| Yazi | MIT |
| lf | MIT |
| fff | MIT |
| xplr | MIT |

Aunque Ranger es el más conocido y usado de la lista, yo personalmente recomiendo Yazi. Es un gestor de archivos muy ligero y fácil de usar. Además, es muy personalizable y tiene muchas funcionalidades.

### Editor/Visor de texto

Al igual que el gestor de archivos, el editor/visor de texto no es una herramienta que se suela asociar con la privacidad y el anonimato, pero es recomendable usar un editor/visor de texto que sea software libre.

| Editor/Visor de texto | Licencia |
|-----------------------|----------|
| Vi | BSD |
| Vim | GPL |
| Neovim | Apache |
| Emacs | GPL |
| Nano | GPL |

Cualquier editor/visor de texto de la lista es una gran opción. Pruébalos y elige el que más te guste y mejor se adapte a tus necesidades.

### Editor/Visor de imagenes

Siguiendo con las recomendaciones de software libre, aquí tienes algunos editores/visores de imágenes.

| Editor/Visor de imagenes | Licencia |
|--------------------------|----------|
| feh | GPL |
| sxiv | GPL |

Yo suelo usar feh, ya que se adapta muy bien con Yazi y es muy fácil de usar.

### Borrador de metadatos

Es importante, sobre todo al compartir archivos, borrar los metadatos de los mismos. A continuación, te dejo una lista de borradores de metadatos a considerar.

| Borrador de metadatos | Licencia |
|-----------------------|----------|
| mat2 | GPL |
| ExifTool | GPL |

### Gestor de contraseñas

Es importante usar contraseñas seguras y una contraseña diferente para cada servicio. Esto puede resultar complicado si tenemos que recordar todas las contraseñas. Por ello, es recomendable usar un gestor de contraseñas. A continuación, te dejo una lista de gestores de contraseñas.

| Gestor de contraseña | Licencia |
|----------------------|----------|
| Bitwarden            | GPL      |
| KeepassXC            | GPL      |
| pass                 | GPL      |

Bitwarden es una gran opción ya que permite crear un servidor de contraseñas en local y acceder a él mediante una extensión o la aplicación propia, sin necesidad de confiar en servidores externos.
Keepass y sus forks son gestores de contraseñas más tradicionales que almacenan las contraseñas en una base de datos local encriptada con una contraseña maestra.
Pass, en cambio, está enfocado en la filosofía Unix y almacena las contraseñas dentro de ficheros de texto plano encriptados con GPG.

### Shell

Aunque la Shell no tiene mucho que ver con el tema de anonimato y privacidad, me parece interesante saber qué opciones tenemos a la hora de elegir una Shell. A continuación, te dejo una lista de Shells a considerar.

| Shell      | Licencia |
|------------|----------|
| sh         | BSD      |
| Bash       | GPL      |
| Fish       | GPL      |
| Zsh        | MIT      |

sh es la Shell más basica y simple de la lista. Funciona y normalmente esta instalada en todos los sistemas Unix-like lo que la hace una buena opción para usuarios que no requieran de muchas funcionalidades y que quieran que sus scripts corran en cualquier sistema.


### Terminal

Al igual que la Shell, la terminal o CLI no están muy relacionadas con el anonimato y la privacidad, pero como usuario de Linux es posiblemente la herramienta que más utilices, por lo que elegir una buena terminal es muy importante. A continuación, te dejo una lista de terminales a considerar.

| Terminal | Licencia |
|----------|----------|
| Alacritty| MIT      |
| Kitty    | MIT      |
| St       | MIT      |
| urxvt    | MIT      |

Si buscas simplicidad y funcionalidad, St es una buena opción.
Por otro lado, si buscas gran personalización y utilizar la GPU, Alacritty y Kitty son las opciones a elegir.

Ademas, estas opciones se pueden complementar con el uso de Tmux o Screen para tener varias sesiones de terminal abiertas a la vez.

A mí, personalmente, me gustan todas y las he probado, así que te recomiendo que las pruebes y elijas la que más te guste y mejor se adapte a tus necesidades.

## Conclusion

Con esta guía, espero haberte ayudado a conseguir un poco de anonimato en la red. Si tienes alguna duda o sugerencia, no dudes en contactar conmigo. Recuerda que la seguridad en internet es un tema muy serio y que debemos ser conscientes de los peligros que nos acechan en la red. Si no tienes conocimientos de informática, te recomiendo que busques ayuda de alguien que los tenga.
