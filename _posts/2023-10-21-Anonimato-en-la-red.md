---
title: Anonimato en la red
description: >-
  Guía para conseguir el anonimato en la red.
author: Xabierland
date: 2023-10-21
categories: [Blog]
tags: [Anonimato, Privacidad, Seguridad]
pin: true
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
- [Tails](https://tails.net/)
- [Whonix](https://www.whonix.org/)
- [Qubes OS](https://www.qubes-os.org/)

### VPS

Un VPS es un servidor virtual privado. Es una máquina virtual que se encuentra en un servidor remoto y que puedes usar para lo que quieras.

A la hora de elegir un proveedor de VPS, es importante que elijas uno que no pida datos personales al registrarte. También es recomendable que permitan el pago con criptomonedas, preferiblemente Monero, y que permita el acceso mediante Tor. Además, es importante que el proveedor no esté ubicado en un país que no proteja la privacidad de sus usuarios. Suiza, Estonia e Islandia son buenas opciones.

A continuación te dejo una lista de proveedores de VPS que cumplen con los requisitos anteriores:

| Servicio | Tor | Cripto |
| --- | --- | --- |
| Privex | X | X |
| Njalla | X | X |
| IncogNet | X | X |
| 1984 | | X |

### VPN

Si estás leyendo este "post" mientras estás conectado a una VPN como NordVPN, ExpressVPN, ProtonVPN, CyberGhostVPN... déjame que salude a la agencia de inteligencia detrás de tu VPN.
Antes de seguir leyendo, hazte un favor y desconéctate de tu VPN para que pueda contarte cómo hacer las cosas bien.

La forma correcta de montar y utilizar una VPN es mediante un VPS. De esta forma, tú eres el dueño del servidor y no tienes que confiar en nadie más. Si además, como hemos visto en el punto anterior, usamos una VPS adquirida de forma anónima, tenemos una VPN mucho más segura y privada que cualquier VPN comercial.

Te dejo enlaces de dos tecnologías para ayudarte a configurar tu VPN:

- [OpenVPN](https://openvpn.net/)
- [WireGuard](https://www.wireguard.com/)

Si aún así deseas usar una VPN comercial, la única opción que puedo recomendar es Mullvad. Es una VPN que no pide datos personales al registrarte, acepta Monero como método de pago y ofrece túneles con resistencia cuántica.

- [Mullvad](https://mullvad.net/)

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
| LibreWolf          | MPL      | GUI |
| Ungoogled Chromium | BSD      | GUI |
| PaleMoon           | MPL      | GUI |
| Iridium            | BSD      | GUI |
| Falkon             | GPL      | GUI |
| Tor Browser        | BSD      | GUI |
| Otter Browser      | GPL      | GUI |
| BadWolf            | BSD      | GUI |
| GNU IceCat         | GPL      | GUI |
| Qutebrowser        | GPL      | GUI |
| w3m                | MIT      | TUI |
| Links              | GPL      | TUI |
| Lynx               | GPL      | TUI |

Firefox es el navegador más conocido de la lista, pero hay que tener en cuenta que no está enfocado en la privacidad ni el anonimato. Es posible configurarlo e instalarle extensiones para "reforzarlo", pero aun así no es la mejor de las opciones.

LibreWolf es un fork de Firefox enfocado en la privacidad y el anonimato. Viene preconfigurado con muchas extensiones y configuraciones para proteger la privacidad y el anonimato del usuario. Yo uso y recomiendo este navegador en caso de no querer complicarse mucho la vida.

w3m, Links y Lynx son navegadores de texto que no soportan JavaScript ni CSS. Son muy útiles para navegar por la web de forma anónima y privada, ya que no permiten que los sitios rastreen al usuario. También son muy útiles para navegar por la web en dispositivos con poca potencia o sin interfaz gráfica.

También es recomendable que al usar cualquier navegador de la lista deshabilitemos JavaScript y las cookies, ya que ambos son métodos muy usados para rastrear a los usuarios y obtener información sobre ellos. Los más celosos de su seguridad en la web buscan, en parte, hacer una vuelta a los orígenes de la web con la Web 1.0 y plantean el uso de XHTML y CSS para hacer nuestras páginas web. Richard Stallman es un gran defensor de esta idea y crítico del uso de JavaScript en la web. [La trampa de JavaScript](https://www.gnu.org/philosophy/javascript-trap.html){:target="_blank"}

#### Extensiones

Si has elegido uno de los navegadores minimalistas de la lista de arriba, la realidad es que no vas a necesitar o no vas a poder usar extensiones. Si, por el contrario, te has visto obligado a elegir una opción como LibreWolf, Ungoogled Chromium o PaleMoon, te recomiendo que eches un vistazo a las siguientes extensiones:

- uBlock Origin
- Bitwareden
- KeepassXC
- Canvas Blocker
- xBrowserSync
- LibreWolf Updater
- Privacy Badger
- User-Agent Switcher
- Chaff
- NoScript

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

Existen muchas más Shells, pero tampoco quiero hacer una lista interminable. Mi preferida es Zsh, ya que es muy personalizable y tiene muchas funcionalidades.

### Terminal

Al igual que la Shell, la terminal o CLI no están muy relacionadas con el anonimato y la privacidad, pero como usuario de Linux es posiblemente la herramienta que más utilices, por lo que elegir una buena terminal es muy importante. A continuación, te dejo una lista de terminales a considerar.

| Terminal | Licencia |
|----------|----------|
| Alacritty| MIT      |
| Kitty    | MIT      |
| St       | MIT      |
| urxvt    | MIT      |
| tmux     | BSD      |

Si buscas simplicidad y funcionalidad, St es una buena opción.
Por otro lado, si buscas gran personalización y utilizar la GPU, Alacritty y Kitty son las opciones a elegir.
Si buscas una terminal multiplexada, Tmux es tu terminal.

A mí, personalmente, me gustan todas y las he probado, así que te recomiendo que las pruebes y elijas la que más te guste y mejor se adapte a tus necesidades.

## Conclusion

Con esta guía, espero haberte ayudado a conseguir el anonimato en la red. Si tienes alguna duda o sugerencia, no dudes en contactar conmigo. Recuerda que la seguridad en internet es un tema muy serio y que debemos ser conscientes de los peligros que nos acechan en la red. Si no tienes conocimientos de informática, te recomiendo que busques ayuda de alguien que los tenga.
