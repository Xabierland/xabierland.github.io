---
title: España bloquea Telegram
author: Xabierland
date: 2024-03-23
categories: [Blog]
tags: [Telegram, Bloqueo, España, Privacidad]
---

## ¿Que ha pasado?

Hoy, 23 de marzo de 2024, el gobierno de España bloquea por orden del juez de la Audiencia Nacional, Santiago Pedraz, la aplicación de mensajería Telegram. Esto se debe a la denuncia de Mediaset, Antena 3 y Movistar por inflingir los derechos de autor.

Esto claramente es una medida desproporcionada y que atenta contra la libertad de expresión y el derecho a la información. Telegram no es solo una aplicación de piratería, es en esencia una aplicación de mensajería que permite a las personas comunicarse de manera segura y privada.

Bloquear Telegram no va a disminuir la piratería, simplemente la va a desplazar a otras aplicaciones o servicios. Lo que realmente se consigue con este bloqueo es coartar la libertad de expresión que ofrece Telegram obligando a los usuarios a usar otras aplicaciones de mensajería que no ofrecen las mismas garantías de privacidad y seguridad y que posiblemente estén mas dispuestas a ceder a las presiones de los gobiernos.

Además que no existe un bloqueo perfecto, siempre habrá maneras de eludirlos, por lo que la gente que realmente usar Telegram para pirateria seguirá haciendolo siendo los únicos afectados los usuarios que usan Telegram para comunicarse.

Esta medida planta un peligroso precedente que puede ser usado en el futuro para bloquear otras aplicaciones o servicios como Signal, Tor, Torrents...
Solo hace falta ver que los paises que han bloqueado Telegram son Rusia, China, Iran y Corea del Norte, paises que no son precisamente ejemplos de democracia y libertad de expresión.

## ¿Que va a pasar ahora?

A nivel de la aplicación, Telegram seguirá funcionando pero no se podrá descargar desde las tiendas de aplicaciones de Google y Apple. Tendremos que descargarla desde la web de Telegram o desde una tienda de aplicaciones de terceros. Esto para dispositivos Android no es un problema, pero para dispositivos iOS si que lo es ya que Apple no permite instalar aplicaciones de terceros sin hacer jailbreak.

A nivel de internet, lo mas probable es que las ISP de España borren de los DNS la dirección de Telegram, lo que hará que no se pueda acceder a la aplicación a priori. Sin embargo, este bloqueo es fácilmente eludible simplemente cambiando los DNS de nuestro ISP por los de Google, Cloudflare, quad9 o incluso montarse uno en un PiHole.

En caso de que el bloqueo no sea a nivel de DNS, sino que bloqueen la IP a nivel de BGP, la solución sería usar una VPN o un proxy para acceder a Telegram.

En cualquier caso, Telegram seguirá funcionando, simplemente habrá que hacer un poco de ingeniería para poder acceder a ella.

## ¿Como puedo solucionarlo?

En el caso de que el bloqueo sea a nivel de DNS, simplemente modificando el DNS bien en el router o en el dispositivo que usemos para acceder a Telegram. Cada dispositivo es un mundo, pero en general, en la configuración de red, en la sección de DNS, podemos poner la dirección que queramos. Aqui tienes una lista de DNS que puedes usar:

| DNS        | IP1     | IP2             | URL                                                                |
| ---------- | ------- | --------------- | ------------------------------------------------------------------ |
| Google     | 8.8.8.8 | 8.8.4.4         | [Google](https://developers.google.com/speed/public-dns?hl=es-419) |
| Cloudflare | 1.1.1.1 | 1.0.0.1         | [Cloudflare](https://developers.cloudflare.com/)                   |
| quad9      | 9.9.9.9 | 149.112.112.112 | [quad9](https://www.quad9.net/)                                    |

En el caso de que el bloqueo sea a nivel de BGP, la solución sería usar una VPN o/y un proxy.

Para VPN, yo recomiendo [Mullvad](https://mullvad.net/es) o una VPN en tu servidor VPS con OpenVPN.

Para proxy, tienes la opción al igual que la VPN de montarte un proxy en una VPS con [MTProxy](https://github.com/TelegramMessenger/MTProxy) o de elegir uno de los servidor de un grupo de activistas como puede ser el caso de [DigitalResistance.dog](https://digitalresistance.dog/) que son un grupo de activistas que luchan por la libertad en internet desde 2018 con el veto de Telegram en Rusia.

En cualquier caso, no te preocupes, siempre habrá maneras de eludir los bloqueos y acceder a Telegram.

## Diccionario

### VPS

> Virtual Private Server. Es un servidor virtual que se aloja en un servidor físico y que se comporta como si fuera un servidor físico independiente. Se usa para alojar aplicaciones y servicios.

### Proxy

> Un proxy es un intermediario entre el usuario y el servidor al que quiere acceder. El proxy recibe la petición del usuario, la procesa y la envía al servidor. El servidor recibe la petición del proxy y la procesa como si fuera una petición directa del usuario.

### ISP

> Internet Service Provider. Es la empresa que nos proporciona acceso a internet. En España, las principales ISP son Movistar, Vodafone y Orange

### DNS

> Domain Name System. Es un sistema de nomenclatura jerárquico descentralizado para dispositivos conectados a redes IP como internet o una red privada. Traduce nombres de dominio a direcciones IP. Como si fuera una agenda de contactos, pero en vez de nombres y números de teléfono, tenemos nombres de dominio y direcciones IP.

### BGP

> Border Gateway Protocol. Es un protocolo de enrutamiento que se utiliza para intercambiar información de enrutamiento entre sistemas autónomos (AS) en internet. Es el protocolo que se usa para intercambiar información de enrutamiento entre los ISP. Si un ISP bloquea una IP a nivel de BGP, esta IP no se podrá acceder desde la red de ese ISP.

### MTProxy

> Es un protocolo de proxy desarrollado por Telegram para eludir los bloqueos de los ISP. Es un protocolo de proxy ligero y seguro que se puede montar en un servidor VPS.
