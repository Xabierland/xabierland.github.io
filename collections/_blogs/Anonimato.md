---
layout: blog
title: Anonimato total en la red
author: Xabier Gabiña Barañano
date: 2023-10-21
---

## Introduccion

La seguridad en internet es algo que ha ido desapareciendo con el paso del tiempo. Atras queda el tiempo en los que la red era un sitio anonimo y descentralizado, en los que todo era por personas y para personas, dando paso a una red practicamente centralizada y vigilada por agencias guvernamentales o grandes compañias de tecnologia. Este post intentara volver atras en el tiempo en algunos aspectos para asi poder avanzar hacia un futuro mas seguro, privado y anonimo.

### Objetivo

El objetivo de este post es el de crear un punto de referencia para todas aquellas personas que quieran empezar a usar la red de forma anonima y privada. Esta guia no es para todo el mundo, si no eres un usuario medianamente avanzado de los ordenadores y no tienes conocimientos de redes, criptografia, sistemas operativos, etc... esta guia no es para ti.

Esta guia puede resultarte util si te dedicas al periodismo y necesitas una cierta anonimidad para tus investigaciones, si perteneces a alguna organizacion activista y buscar reinvidicar tus derechos, si eres un whistleblower y quieres denunciar algun tipo de corrupcion o simplemente si eres un usuario con conocimientos y conciencia que quiere recuperar su privacidad en la red.

### Disclaimer

Antes de empezar con la guia quiero dejar claro que mis conocimientos de la materia no son absolutos. Todo lo que voy a explicar en este post es lo que yo he aprendido con el tiempo y con la experiencia. Si alguien ve algun error o tiene alguna sugerencia que hacerme no dudeis en contactar conmigo.

## Hardware

El Hardware aunque sea lo primero que vamos a tratar en este post es la parte menos importante y a la vez mas dificil de conseguir de forma anonima. Mi recomendacion en caso de que estes muy interesado en obtener el hardware de forma anonima es:

Comprar el hardware en una tienda fisica en efectivo. No es la mejor forma ya que en dinero fiat es rastreable pero es la forma mas facil de conseguirlo.

Comprar el hardware en una tienda con una tarjeta de regalo anonima. Te en cuenta que no puedes dar datos personales a la hora de comprar con este metodo ya que no serviaria de nada.

Comprar el hardware online mediante criptomonedas. Esta es la forma mas dificil de conseguirlo ya que no hay muchas tiendas que acepten criptomonedas y las que lo hacen suelen pedir datos personales.

Comprar el hardware online de segunda mano. Depende mucho de la persona que te lo venda y es complicado encontrar a alguien que no te pida datos personales, acepte criptomonedas y que lo envie de forma anonima a la vez que no te estafe.

Una vez visto los metodos de adquisicion que yo conozco pasemos a ver que hardware es el mas adecuado para conseguir el anonimato total en la red.

### Ordenador de sobremesa

El ordenador de sobremesa es quizas el mas facil de conseguir ya que puedes ir comprando las piezas poco a poco en diferentes tiendas y montarlo tu mismo en casa. Tambien tiene como ventaja que suelen por lo general ser mas potentes que un ordenador portatil y por lo tanto mas utiles para tareas que requieran de mas potencia. Tambien es facil de modificar y ampliar.

Como contraparte tenemos que es un ordenador que no es facil de transportar y que ocupa bastante espacio. Al estar en un mismo sitio es mas facil que pueda ser localizado. Tambien es mas dificil de ocultar en caso de que sea necesario.

### Ordenador portatil

El ordenador portatil es quizas el mas dificil de conseguir de forma anonima ya que por lo general se suele comprar en una tienda fisica o online y se suele pagar con tarjeta de credito o debito. Tambien es posible comprarlo en efectivo pero es mas dificil de encontrar una tienda que acepte efectivo que una que acepte tarjeta.

Es recomendable elegir un ordenador sin sistema operativo ya que asi nos ahorramos el tener que formatear el disco duro y nos aseguramos de que no hay spyware preinstalado.

Entre sus ventajas tenemos su gran movilidad y adaptavilidad. Es facil de transportar y ocultar en caso de que sea necesario.

Por otro lado sacrificamos la potencia y la capacidad de modificacion.

### Dispositivo Movil

Dentro de los dispositivos moviles englobo tanto telefonos como tables. Estas por lo general suelen venir con Android por lo que es recomendable cambiar el sistema operativo por uno mas seguro y privado como puede ser  GrapheneOS. Tambien es posible encontrar moviles ya fabricados con otros forks de Android con capas de seguridad aunque al estar fabricados por empresas privadas no se puede garantizar que no tengan spyware.

El dispositivo movil es algo muy importante ya que es el dispositivo con mas uso en el dia a dia y que siempre llevamos con nosotros. Si no tenemos cuidado con el puede ser un gran punto de fuga de informacion.

## Software

Cuando hablamos de Software anonimo y privado es imposible no hablar de FOSS. FOSS es el acronimo de Free and Open Source Software. Es un movimiento que busca la libertad de los usuarios y la transparencia de los programas. Es por ello que es esta seccion voy a hablar de tipos de software y todos ellos van a ser FOSS.

### OS

Un sistema operativo es un conjunto de programas que permiten la comunicacion entre el usuario y el hardware.

Elegir un sistema operativo es muy complicado, hay muchas opciones y cada una tiene sus ventajas y desventajas. Ademas hay mucho "fandom" en torno a los sistemas operativos y es dificil encontrar una opinion objetiva sobre ellos, esta incluida.

Para elegir un sistema operativo, yo, intentaria buscar uno que sea libre y que no tenga una empresa detras que pueda tener intereses en recopilar informacion sobre ti. Tambien es importante que el sistema operativo sea lo mas transparente posible y que no tenga programas preinstalados que puedan ser usados para recopilar informacion sobre ti.

Como no quiero meterme en camisas de once varas y no quiero que este post se convierta en una discusion sobre que sistema operativo es mejor o peor, voy a dejar una lista de sistemas operativos que cumplen con los requisitos anteriores y que son buenas opciones a la hora de elegir un sistema operativo.

- [Debian](https://www.debian.org/)
- [Arch Linux](https://www.archlinux.org/)
- [Gentoo](https://www.gentoo.org/)
- [OpenBSD](https://www.openbsd.org/)
- [Qubes OS](https://www.qubes-os.org/)
- [Whonix](https://www.whonix.org/)
- [Linux From Scratch](https://www.linuxfromscratch.org/)

### VPS

Un VPS es un servidor virtual privado. Es una maquina virtual que se encuentra en un servidor remoto y que puedes usar para lo que quieras.

A la hora de elegir un proveedor de VPS es importante que elijas uno que no te pida datos personales a la hora de registrarte. Es tambien bueno que permitan el pago con criptomonedas prefereblemente Monero y que permita el acceso mediante Tor. Ademas es importante que el proveedor no este ubicado en un pais que no proteja la privacidad de sus usuarios. Suiza, Estonia e Islandia son buenas opciones.

A continuacion te dejo una lista de proveedores de VPS que cumplen con los requisitos anteriores:

| Servicio | Tor | Cripto |
| --- | --- | --- |
| Privex | X | X |
| Njalla | X | X |
| IncogNet | X | X |
| 1984 | | X |

### VPN

Si estas leyendo este "post" mientras estas conectado a una VPN como puede ser NordVPN, ExpressVPN, ProtonVPN, CyberGhostVPN... dejame que salude a la agencia de inteligencia detras de tu VPN.
Antes de seguir leyendo, hazte un favor y deconectate de tu VPN para que te pueda contar como hacer las cosas bien.

La forma correcta de montar y utilizar un VPN es mediante un VPS. De esta forma tu eres el dueño del servidor y no tienes que confiar en nadie mas. Si ademas, como hemos visto en el punto anterior usamos una VPS adquirida de forma anonima, tenemos una VPN mucho mas segura y privada que cualquier VPN comercial.

Te dejo links de dos tecnologias para ayudarte a configurar tu VPN:

- [OpenVPN](https://openvpn.net/)
- [WireGuard](https://www.wireguard.com/)

### Proxy

Antes de hablar de Proxys quiero desmentir el mito de que es inseguro usar una VPN con Proxys. La razon por la que se desaconseja una VPN con Proxys es para cuando usas un VPN comercial ya que estas pueden llegar a recopilar informacion sobre este tipo de conexiones. Si usas una VPN montada por ti mismo en una VPS no hay problema en usar Proxys.

Dicho esto vamos a hablar de Proxys.

### Navegador

Los navegadores son una herramienta crucial y que usamos a todas horas por lo que elegir uno no es *moco de pavo*. A continuacion te dejo una lista de navegadores a tener en cuenta a la hora de elegir el tuyo.

| Navegador          | Licencia  |
|--------------------|-----------|
| Mozilla Firefox    | MPL       |
| LibreWolf          | MPL       |
| Ungoogled Chromium | BSD       |
| PaleMoon           | MPL       |
| Iridium            | BSD       |
| Falcon             | GPL       |
| Netsurf            | GPL       |
| Tor Browser        | BSD       |
| Otter Browser      | GPL       |
| BadWolf            | BSD       |
| GNU IceCat         | GPL       |
| Qutebrowser        | GPL       |
| Lynx               | GPL       |
| vimb               | GPL       |
| Surf               | MIT       |
| webbrowser.git     | GPL       |

LibreWolf Ungoogle Chromium y PaleMoon no son malas opciones y estan disponibles para todos los sistemas operativos pero la mayoria de ellos traen una cantidad de funcionalidad que no vamos a usar y que son un potencial riesgo para nuestra privacidad ademas de lastrar el rendimiento. Es por ello que si buscamos la privacidad debemos buscar navegadores minimalistas como puede ser webbrowser. Tor Browser seria un caso aparte ya que esta especialmente diseñado para ser usado cuando se navega por la red Tor.

Tambien es recomendable que al usar cualquier navegador de la lista desbilitemos JavaScript y las cookies. Ambos son metodos muy usados para rastrear a los usuarios y obtener informacion sobre ellos. Es por ello que los mas recelosos de su seguridad en la web buscan en parte hacer una vuelta atras a los origenes de la web con Web 1.0 y plantenado el uso de XHTML y CSS para hacer nuestras paginas webs. Richar Stallman es un gran defensor de esta idea y critico duramente el uso de JavaScript en la web. [La trampa de JavaScript](https://www.gnu.org/philosophy/javascript-trap.html){:target="_blank"}

Especial mencion al proyecto [pipe-viewer](https://github.com/trizen/pipe-viewer){:target="_blank"} un proyecto de GitHub que permite buscar y reproducir videos de YouTube sin necesidad de usar JavaScript.

#### Extensiones

Si has elegido uno de los navegadores minimalistas de la lista de arriba, la realidad es que no vas a necesitar o no vas a poder usar extensiones. Si por el contrario te has visto obligado a elegir una opcion como LibreWolf, Ungoogle Chromium o PaleMoon te recomiendo que heches un vistazo a las siguientes extensiones:

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

### Dominio

### Correo

### Mensajeria

Voy a ir directo al grano. Si quieres privacidad y anonimato olvidate de WhatsApp, Telegram, Signal, Discord, Slack... Todas estas aplicaciones estan obligadas por ley a tener que dar informacion sobre sus usuarios a las agencias de inteligencia además de que no son software libre y por lo tanto no se puede garantizar que no tengan spyware.

La mejor opcion es usar IRC. IRC es un protocolo de comunicacion que permite la comunicacion entre usuarios mediante texto plano. Es un protocolo muy antiguo y que no ha cambiado mucho desde su creacion. Es un protocolo muy simple y que no tiene muchas funcionalidades pero es muy seguro y privado. Ademas es software libre y hay muchos clientes de IRC disponibles para todos los sistemas operativos.

Otras opciones son XMPP y Matrix. Ambos son protocolos de comunicacion que permiten la comunicacion entre usuarios mediante texto plano. Son protocolos mas modernos que IRC y tienen mas funcionalidades pero tambien son mas complejos y por lo tanto mas dificiles de usar. Aun asi IRC sigue siendo la mejor opcion en cuanto a privacidad y anonimato.

### Criptomonedas

Aunque en general el concepto de criptomoneda es bastante seguro y privado, hay ciertas criptomonedas que son mas privadas que otras. Yo, personalmente, recomiendo Monero. Es una criptomoneda que tiene un gran anonimato y privacidad. Ademas es una criptomoneda que esta en constante desarrollo y que tiene una gran comunidad detras. Tambien es una buena opcion Zcash aunque no es tan privada como Monero.

- [Monero](https://www.getmonero.org/)
- [Zcash](https://z.cash/)

### Gestor de archivos

| Gestor de archivos | Licencia |
|--------------------|----------|
| Ranger | GPL |
| Yazi | MIT |
| lf | MIT |
| fff | MIT |
| xplr | MIT |

### Editor/Visor de texto

| Editor/Visor de texto | Licencia |
|-----------------------|----------|
| Vi | BSD |
| Vim | GPL |
| Neovim | Apache |
| Emacs | GPL |

### Editor/Visor de imagenes

| Editor/Visor de imagenes | Licencia |
|--------------------------|----------|
| feh | GPL |
| sxiv | GPL |

### Editor/Visor de PDF

| Editor/Visor de PDF | Licencia |
|---------------------|----------|
| Zathura | GPL |

### Borrador de metadatos

| Borrador de metadatos | Licencia |
|-----------------------|----------|
| mat2 | GPL |

### Gestor de contraseñas

Es importante usar contraseñas seguras ademas de usar una contraseña diferente para cada servicio. Esto puede resultar complicado si tenemos que acordarnos de todas las contraseñas. Es por ello que es recomendable usar un gestor de contraseñas. A continuacion te dejo una lista de gestores de contraseñas a tener en cuenta a la hora de elegir el tuyo.

| Gestor de contraseña | Licencia |
|----------------------|----------|
| Bitwarden            | GPL      |
| Keepass              | GPL      |
| KeepassX             | GPL      |
| KeepassXC            | GPL      |
| pass                 | GPL      |

Bitwarden es una gran opcion ya que permite crear un servidor de contraseñas en local y acceder a el mediante un extension o la aplicacion propia. De esta forma no tenemos que confiar en ningun servidor externo para almacenar nuestras contraseñas.
Keepass y sus forks son gestores de contraseñas mas tradiciones que almacenan las contraseñas en una base de datos local escriptada con una contraseña maestra.
pass en cambio, esta enfocado en la filosofia Unix y almacena las contraseñas dentro de ficheros de texto plano encriptados con GPG.

### Shell

Aunque la Shell no tiene mucho que ver con el tema de anonimato y privacidad me parece interesante saber que opciones tenemos a la hora de elegir una Shell. A continuacion te dejo una lista de Shells a tener en cuenta a la hora de elegir la tuya.

| Shell      | Licencia |
|------------|----------|
| Bash       | GPL      |
| Fish       | GPL      |
| Zsh        | MIT      |

Mi preferida es Zsh ya que es muy personalizable, muchas funcionalidades y trae mucha personalizacion.

### Terminal

Al igual que la Shell, la terminal o cli, no estan muy relacionados con el anonimato y privacidad pero como usuario de Linux es posiblemente la herramienta que mas utilices por lo que elegir una buena terminal es muy importante. A continuacion te dejo una lista de terminales a tener en cuenta a la hora de elegir la tuya.

| Terminal | Licencia |
|----------|----------|
| Alacritty| MIT      |
| Kitty    | MIT      |
| St       | MIT      |

Si buscas la simplicidad y funcionalidad st es tu terminal.
Por el otro lado, si buscas gran personalizacion y utilizar GPU Alacritty y Kitty son las opciones a elegir.
