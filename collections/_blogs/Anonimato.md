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

Dentro de los dispositivos moviles englobo tanto telefonos como tables. Estas por lo general suelen venir con Android por lo que es recomendable cambiar el sistema operativo por uno mas seguro y privado como puede ser LineageOS o GrapheneOS. Tambien es posible encontrar moviles ya fabricados con otros forks de Android con capas de seguridad aunque al estar fabricados por empresas privadas no se puede garantizar que no tengan spyware.

El dispositivo movil es algo muy importante ya que es el dispositivo con mas uso en el dia a dia y que siempre llevamos con nosotros. Si no tenemos cuidado con el puede ser un gran punto de fuga de informacion.

## Software

### OS

Elegir un buen sistema operativo es un punto decisivo a la hora de conseguir el anonimato en la red. Si eres usuario de Mac o Windows vete haciendo a la idea de que te va a tocar formatear tu ordenador para instalar un sistema operativo **de verdad**.

A continuacion te dejo una lista de sistemas operativos a tener en cuenta a la hora de elegir el tuyo. Los he ordenado segun mi criterio de peor a mejor.

| Distro     | Licencia | Privacidad | Dificultad   | Opinion                                                                                                                                                                                       |
|------------|----------|------------|--------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Ubuntu     | GPL      | Ninguna    | Principiante | Si la privacidad y el anonimato te importan no lo uses, posiblemente Amazon y otras empresas reciban informacion de Ubuntu.                                                                   |
| Pop!_OS    | GPL      | Ninguna    | Principiante | Si Ubuntu tenia poca esta todavia tiene menos.                                                                                                                                                |
| Deepin     | GPL      | Ninguna    | Principiante | Si quieres que los chinos sepan todo sobre ti para ir teniendo un poco de credito social te lo recomiendo.                                                                                    |
| Fedora     | GPL      | Ninguna    | Principiante | Si en vez de ser fan de china lo eres de Estados Unidos instala Fedora, asi la NSA leera lo mucho que te gusta su pais.                                                                       |
| Linux Mint | GPL      | Buena      | Principiante | No lo usaria aunque es una buena opcion para empezar en linux si no tienes ni idea.                                                                                                           |
| OpenSUSE   | GPL      | Buena      | Principiante | Aunque sigo desconfiando de el no he encontrado ningun indicio de spyware a diferencia de los ya nombrados.                                                                                   |
| Kali Linux | GPL      | Buena      | Principiante | Buena cantidad de herramientas utiles para el hacking pero no le veo futuro como sistema operativo para todos los dias.                                                                       |
| Parrot OS  | GPL      | Buena      | Principiante | Lo mismo que Kali Linux.                                                                                                                                                                      |
| Tails      | GPL      | Excelente  | Principiante | Si lo que buscas es un sistema de escritorio esto no es para ti. Basicamente su unica funcion es la de navegar por la red Tor y que al apagar todo se olvide.                                 |
| Red Hat    | GPL      | ?          | Intermedia   | Nunca lo he probado asi que no puedo hablar mucho de el pero la premisa de pagar por usarlo ya me hecha para atras.                                                                           |
| Manjaro    | GPL      | Muy buena  | Intermedia   | Esta basado en Arch pero con la idea de que sea mas facil para el usuario sin tanta experiencia. Si te asusta usar Arch no es una opcion horrible pero teniendo Arch no le veo mucho sentido. |
| Debian     | GPL      | Muy buena  | Intermedia   | Es el padre de practicamente todos los sistemas de esta lista. Por ello es bastante moldeable y no una mala opcion si vienes de otro sistema basado en Debian.                                |
| OpenBSD    | BSD      | Excelente  | Avanzado     | Una gran opcion, sin programas inutiles, transparentes, licencias publicas. La unica pega es que no tiene toda la comunidad que puede tener Linux.                                            |
| Qubes OS   | GPL      | Excelente  | Avanzada     | Bajo la premisa de que nada es seguro la idea de este OS es la de encapsuar en maquinas virtuales todas las aplicaciones para que no interactuen con el resto y se mantengan seguras.         |
| Arch Linux | GPL      | Excelente  | Avanzada     | Simplemente el equilibrio perfecto entre privacidad, anonimato, usabilidad y personalizacion sin llegar a los extemos de Gentoo o LFS.                                                        |
| Gentoo     | GPL      | Excelente  | Dificil      | Un paso inferior a LFS pero con la misma idea de si quieres algo haztelo tu mismo.                                                                                                            |
| LFS        | MIT      | Excelente  | Maxima       | Si tienes lo que hace falta para montarte la distro compilando todo desde el src este amigo mio es tu sistema.                                                                                |

Dando por hecho que si estas leyendo este post solo buscas la excelencia, mi recomendacion personal es Arch Linux. Es un sistema operativo que te permite hacer lo que quieras con el y que no te pone limites. Es excelente en todos los aspectos y si tienes un poco de conocimientos de linux no te resultara muy dificil de usar.

La otra opcion por la que me decantaria es Qubes OS usando Whonix para redirigir el trafico mediante la red TOR. Este sistema operativo es excelente en cuanto a seguridad y privacidad pero es mas dificil de usar que Arch Linux y no es tan personalizable.

Tambien decir que Kali Linux, Parrot OS y Tails son sistemas operativos que aunque no esten muy enfocados en ser la base de un sistema operativo de escritorio son muy utiles para ciertas tareas como puede ser el hacking o la navegacion anonima. Por lo que usarlos como maquinas virtuales en Qubes OS o Arch Linux es una buena opcion.

### VPN

Si estas leyendo este "post" mientras estas conectado a una VPN como puede ser NordVPN, ExpressVPN, ProtonVPN, CyberGhostVPN... dejame que salude a la agencia de inteligencia detras de tu VPN.
Antes de seguir leyendo, hazte un favor y deconectate de tu VPN para que te pueda contar como hacer las cosas bien.

La forma correcta de montar y utilizar un VPN es mediante un VPS. Yo personalmente te recomiendo que busques alguna en Estonia o Suiza ya que son paises que tienen leyes muy estrictas en cuanto a la privacidad de los datos de sus ciudadanos y por lo tanto es mas dificil que te puedan obligar a dar informacion sobre ti.
Un vez que tengas tu VPS monta tu VPN usando WireGuard o OpenVPN y conectate a ella desde cualquier dispositivo que quieras usar de forma anonima.

Te dejo links de ambas tecnologias para ayudarte a configurar tu VPN:

* [WireGuard](https://www.wireguard.com/)
* [OpenVPN](https://openvpn.net/)

### Proxy

### Navegador

Los navegadores son una herramienta crucial y que usamos a todas horas por lo que elegir uno no es *moco de pavo*. A continuacion te dejo una lista de navegadores a tener en cuenta a la hora de elegir el tuyo. Los he ordenado segun mi criterio de peor a mejor.

| Navegador          | Licencia  | Privacidad | Opinion                                                                                                                 |
|--------------------|-----------|------------|-------------------------------------------------------------------------------------------------------------------------|
| Safari             | Privativa | Ninguna    | ¿De verdad hace falta que de mi opinion?                                                                                |
| Microsoft Edge     | Privativa | Ninguna    | ¿Alguien sabe como hago apt remove Edge?                                                                                |
| Google Chrome      | Freeware  | Ninguna    | Salir de Chrome es parte de madurar.                                                                                    |
| Opera              | Freeware  | Ninguna    | Solo con ver lo que invierten en publicidad siendo Freeware ya dice mucho.                                              |
| Vivaldi            | Freeware  | Ninguna    | Ex-trabajadores de Opera intentando hacer lo mismo.                                                                     |
| Mozilla Firefox    | MPL       | Mala       | Intenta hacer las cosas bien pero sin hacer trampas no puedes seguir el ritmo a Chrome.                                 |
| Brave              | MPL       | Mala       | Vamos avanzando, una pena que usen el motor de renderizado de Google.                                                   |
| LibreWolf          | MPL       | Buena      | Es una buena opcion si estas obligado a usar la Web 3.0 y JS                                                            |
| Ungoogled Chromium | BSD       | Buena      | Lo mismo que LibreWolf aunque por el tema de la licencia me convence mas.                                               |
| PaleMoon           | MPL       | Buena      | Si por alguna razon estas obligado a usar Windows o Mac sin duda la mejor opcion para ellos.                            |
| Iridium            | BSD       | Muy Buena  | Chromium pero con asteroides                                                                                            |
| Falcon             | GPL       | Muy Buena  | Ligero y bastante seguro. No es mala opcion                                                                             |
| Netsurf            | GPL       | Muy Buena  | Un navegador muy ligero, portable y seguro pero con peor uso que Falcon.                                                |
| Tor Browser        | BSD       | Muy Buena  | No es un navegador para todo el dia. Su uso deberia ser exclusivo para navegar por la red Tor.                          |
| Otter Browser      | GPL       | Excelente  | Es un navegador privado que intenta asimilarse a Opera en la antiguedad. No me llama mucho pero esta bien conocerlo.    |
| BadWolf            | BSD       | Excelente  | Gran opcion, privada manteniendo la usabilidad que se suele sacrificar con las opciones mas privadas.                   |
| GNU IceCat         | GPL       | Excelente  | Navegador muy interesante con una implementacion diferente de JS para minimizar el rastreo.                             |
| Qutebrowser        | GPL       | Excelente  | Navegador que funciona mediante atajo de teclados. Minimalista, seguro y si lo dominas rapido.                          |
| Lynx               | GPL       | Excelente  | Aunque he de admitir que no es el mas utilizable por el simple hecho de ser CLI tiene un puesto en mi corazon.          |
| vimb               | GPL       | Excelente  | ¿Te gusta vim? Pues esto es lo mismo pero como navegador. Idea similar a Qutebrowser pero con interfaz mas minimalista, |
| Surf               | MIT       | Excelente  | Junto con Lynx y vimb es el minimalismo y la configurabilidad al limite.                                                |
| webbrowser.git     | GPL       | Excelente  | Navegador web basico pero muy efectivo. Equilibrio entre privacidad, minimalismo y usabilidad.                          |

El principal momento al elegir navegador es que cada uno le da un uso muy particular. Si lo que te interesa es la privacidad los siete primeros no son una opcion viable. Brave esta en un punto muerto en que aun teniendo buenas practicas respecto a los puestos superiores de la lista no llega a los estandares de privacidad que buscamos por lo que no puedo recomendarlo en su totalidad. LibreWolf Ungoogle Chromium y PaleMoon no son malas opciones y estan disponibles para todos los sistemas operativos pero la mayoria de ellos traen una cantidad de funcionalidad que no vamos a usar y que son un potencial riesgo para nuestra privacidad. Es por ello que si buscamos la privacidad debemos buscar navegadores minimalistas como puede ser webbrowser. Tor Browser seria un caso aparte ya que esta especialmente diseñado para ser usado cuando se navega por la red Tor.

Tambien es recomendable que al usar cualquier navegador de la lista desbilitemos JavaScript y las cookies. Ambos son metodos muy usados para rastrear a los usuarios y obtener informacion sobre ellos. Es por ello que los mas recelosos de su seguridad en la web buscan en parte hacer una vuelta atras a los origenes de la web con Web 1.0 y plantenado el uso de XHTML y CSS para hacer nuestras paginas webs. Richar Stallman es un gran defensor de esta idea y critico duramente el uso de JavaScript en la web. [La trampa de JavaScript](https://www.gnu.org/philosophy/javascript-trap.html)

Especial mencion al proyecto [pipe-viewer](https://github.com/trizen/pipe-viewer) un proyecto de GitHub que permite buscar y reproducir videos de YouTube sin necesidad de usar JavaScript.

#### Extensiones

Si has elegido uno de los navegadores minimalistas de la lista de arriba, la realidad es que no vas a necesitar o no vas a poder usar extensiones. Si por el contrario te has visto obligado a elegir una opcion como LibreWolf, Ungoogle Chromium o PaleMoon te recomiendo que heches un vistazo a las siguientes extensiones:

* uBlock Origin
* Bitwareden
* KeepassXC
* Canvas Blocker
* xBrowserSync
* LibreWolf Updater
* Privacy Badger
* User-Agent Switcher
* Chaff

### Dominio

### Correo

#### Servidor de correo

#### Cliente de correo

### Mensajeria

### Criptomonedas

### Gestor de archivos

### Editor/Visor de texto

### Editor/Visor de imagenes

### Editor/Visor de PDF

### Editor/Visor de RSS

### Borrador de metadatos

### Gestor de contraseñas

| Gestor de contraseña | Licencia | Opinion |
|----------------------|----------|---------|
| LastPass             | Freemium |         |
| Dashlane             | Freemium |         |
| Zoho Vault           | Freemium |         |
| Keeper               | Freemium |         |
| RoboForm             | Freemium |         |
| Sticky Password      | Freemium |         |
| Enpass               | Freemium |         |
| 1Password            | Freemium |         |
| Bitwarden            | GPL      |         |
| passage              | GPL      |         |
| Keepass              | GPL      |         |
| KeepassX             | GPL      |         |
| KeepassXC            | GPL      |         |
| pass                 | GPL      |         |

### Shell

Aunque la Shell no tiene mucho que ver con el tema de anonimato y privacidad me parece interesante saber que opciones tenemos a la hora de elegir una Shell. A continuacion te dejo una lista de Shells a tener en cuenta a la hora de elegir la tuya.

| Shell      | Licencia |
|------------|----------|
| PowerShell | MS-PL    |
| Bash       | GPL      |
| Nu Shell   | Apache   |
| Glsh       | MIT      |
| BusyBox    | GPL      |
| Elvish     | MIT      |
| rc         | MIT      |
| Ash        | BSD      |
| Mksh       | BSD      |
| Dash       | BSD      |
| C Shell    | BSD      |
| Korn Shell | CDDL     |
| Tcsh       | BSD      |
| Fish       | GPL      |
| Zsh        | MIT      |

Quitando PowerShell cualquiera es una buena opcion. Personalmente recomiendo Zsh pero con la que te sientas mas como esta bien.

### Terminal

Al igual que la Shell, la terminal o cli, no estan muy relacionados con el anonimato y privacidad pero como usuario de Linux es posiblemente la herramienta que mas utilices por lo que elegir una buena terminal es muy importante. A continuacion te dejo una lista de terminales a tener en cuenta a la hora de elegir la tuya.
