---
title: España bloquea Telegram
author: Xabierland
date: 2024-03-23
categories: [Blog]
tags: [Telegram, Bloqueo, España, Privacidad]
---

## ¿Qué ha pasado?

Hoy, 23 de marzo de 2024, el gobierno de España ha bloqueado, por orden del juez de la Audiencia Nacional, Santiago Pedraz, la aplicación de mensajería Telegram. Esto se debe a la denuncia de Mediaset, Antena 3 y Movistar por infringir los derechos de autor.

Esto, claramente, es una medida desproporcionada y que atenta contra la libertad de expresión y el derecho a la información. Telegram no es solo una aplicación de piratería; en esencia, es una aplicación de mensajería que permite a las personas comunicarse de manera segura y privada.

Bloquear Telegram no va a disminuir la piratería, simplemente la desplazará a otras aplicaciones o servicios. Lo que realmente se consigue con este bloqueo es coartar la libertad de expresión que ofrece Telegram, obligando a los usuarios a utilizar otras aplicaciones de mensajería que no ofrecen las mismas garantías de privacidad y seguridad, y que posiblemente estén más dispuestas a ceder ante las presiones de los gobiernos.

Además, no existe un bloqueo perfecto: siempre habrá maneras de eludirlos. Por lo tanto, las personas que realmente utilizan Telegram para piratería seguirán haciéndolo, siendo los únicos afectados los usuarios que lo emplean para comunicarse.

Esta medida sienta un peligroso precedente que puede ser utilizado en el futuro para bloquear otras aplicaciones o servicios como Signal, Tor, *torrents*...  
Solo hace falta ver qué países han bloqueado Telegram: Rusia, China, Irán y Corea del Norte, naciones que no son precisamente ejemplos de democracia y libertad de expresión.

## ¿Qué va a pasar ahora?

A nivel de la aplicación, Telegram seguirá funcionando, pero no se podrá descargar desde las tiendas de aplicaciones de Google y Apple. Tendremos que descargarla desde la web de Telegram o desde una tienda de aplicaciones de terceros. Esto, para dispositivos Android, no supone un problema, pero para dispositivos iOS sí lo es, ya que Apple no permite instalar aplicaciones de terceros sin hacer *jailbreak*.

A nivel de internet, lo más probable es que las ISP de España eliminen de sus DNS la dirección de Telegram, lo que impedirá acceder a la aplicación a priori. Sin embargo, este bloqueo es fácilmente eludible simplemente cambiando los DNS de nuestro proveedor por los de Google, Cloudflare, Quad9, o incluso montando uno propio en un Pi-hole.

En caso de que el bloqueo no sea a nivel de DNS, sino que se bloquee la IP a nivel de BGP, la solución sería usar una VPN o un proxy para acceder a Telegram.

En cualquier caso, Telegram seguirá funcionando; simplemente habrá que hacer un poco de ingeniería para poder acceder a la aplicación.

## ¿Cómo puedo solucionarlo?

En el caso de que el bloqueo sea a nivel de DNS, basta con modificar el DNS, ya sea en el router o en el dispositivo que utilicemos para acceder a Telegram. Cada dispositivo es diferente, pero en general, en la configuración de red, en la sección de DNS, podemos introducir la dirección que deseemos. Aquí tienes una lista de DNS que puedes utilizar:

| DNS        | IP1     | IP2             | URL                                                                |
| ---------- | ------- | --------------- | ------------------------------------------------------------------ |
| Google     | 8.8.8.8 | 8.8.4.4         | [Google](https://developers.google.com/speed/public-dns?hl=es-419) |
| Cloudflare | 1.1.1.1 | 1.0.0.1         | [Cloudflare](https://developers.cloudflare.com/)                   |
| Quad9      | 9.9.9.9 | 149.112.112.112 | [Quad9](https://www.quad9.net/)                                    |

En el caso de que el bloqueo sea a nivel de BGP, la solución sería utilizar una VPN y/o un proxy.

Para VPN, recomiendo [Mullvad](https://mullvad.net/es) o instalar tu propia VPN en un servidor VPS con OpenVPN.

Para proxy, tienes la opción, al igual que con la VPN, de montar un proxy en una VPS con [MTProxy](https://github.com/TelegramMessenger/MTProxy), o utilizar uno de los servidores de un grupo de activistas como [DigitalResistance.dog](https://digitalresistance.dog/), que luchan por la libertad en internet desde 2018, tras el veto a Telegram en Rusia.

En cualquier caso, no te preocupes: siempre habrá maneras de eludir los bloqueos y seguir accediendo a Telegram.

## Diccionario

### VPS

> *Virtual Private Server*. Es un servidor virtual que se aloja en un servidor físico y que se comporta como si fuera un servidor físico independiente. Se utiliza para alojar aplicaciones y servicios.

### Proxy

> Un proxy es un intermediario entre el usuario y el servidor al que quiere acceder. El proxy recibe la petición del usuario, la procesa y la envía al servidor. El servidor recibe la petición del proxy y la procesa como si fuera una petición directa del usuario.

### ISP

> *Internet Service Provider*. Es la empresa que nos proporciona acceso a internet. En España, las principales ISP son Movistar, Vodafone y Orange.

### DNS

> *Domain Name System*. Es un sistema de nomenclatura jerárquico descentralizado para dispositivos conectados a redes IP, como internet o una red privada. Traduce nombres de dominio a direcciones IP. Es como una agenda de contactos, pero en lugar de nombres y números de teléfono, tenemos nombres de dominio y direcciones IP.

### BGP

> *Border Gateway Protocol*. Es un protocolo de enrutamiento que se utiliza para intercambiar información de enrutamiento entre sistemas autónomos (AS) en internet. Es el protocolo que utilizan las ISP para comunicarse entre sí. Si un ISP bloquea una IP a nivel de BGP, esa IP no será accesible desde su red.

### MTProxy

> Es un protocolo de proxy desarrollado por Telegram para eludir los bloqueos de las ISP. Es un protocolo de proxy ligero y seguro que se puede montar en un servidor VPS.
