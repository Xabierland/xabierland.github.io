---
title: OpenVPN para tu casa
author: Xabierland
description: >-
  Instala y configura OpenVPN en tu casa para acceder a tu red local de forma segura y privada desde cualquier lugar.
date: 2024-11-09
categories: [Tutorial, HomeLab]
tags: [OpenVPN, VPN, Redes, Seguridad, Proxmox]
---

## Introducción

En ocasiones, necesitamos acceder a nuestra red doméstica cuando estamos fuera de casa. Ya sea para acceder a archivos, controlar aplicaciones y dispositivos o simplemente para navegar de forma segura y privada, una VPN es una solución ideal para conectarse de forma segura a nuestra red doméstica desde cualquier lugar.

### ¿Qué es OpenVPN?

OpenVPN es un software de código abierto que permite crear una red privada virtual (VPN) segura y cifrada. OpenVPN es una de las soluciones de VPN más populares y ampliamente utilizadas en el mundo debido a su facilidad de uso, seguridad y flexibilidad.

OpenVPN es compatible con la mayoría de los sistemas operativos, incluyendo Windows, macOS, Linux, Android e iOS.

### ¿OpenVPN o WireGuard?

WireGuard es una tecnología de VPN más moderna y durante mucho tiempo mas rápida que OpenVPN. Sin embargo, OpenVPN ha mejorado enormemente con la llegada de DCO (Data Channel Offload)

DCO cambia la forma en que Access Server maneja los datos que fluyen a través del túnel VPN. Con DCO, el cifrado y descifrado del canal de datos se trasladan al espacio del kernel, permitiendo que sea el kernel quien realice el trabajo en lugar de gestionarlo en el espacio de usuario. Esto ahorra en operaciones de copia entre el espacio del kernel y el espacio de usuario y utiliza la multihilación.[^1]

Esto combinado con la facilidad de uso, la madurez, la compatibilidad y la seguridad extra que ya tenia OpenVPN sobre WireGuard, hacen que para mi, OpenVPN sea la mejor opción a día de hoy.

## Instalación y configuración de OpenVPN [^2]

> Este tutorial asume que tienes un servidpr Proxmox en tu casa y que tienes conocimientos básicos de Linux y redes.
> Si no tienes un servidor Proxmox, puedes seguir este [tutorial](https://xabierland.github.io/posts/Monta-un-servidor-con-Proxmox/)
> Si no tienes conocimientos de Linux y redes te recomiendo formarte antes de seguir con este tutorial.
{: .prompt-warning }

### 1. Crear un contenedor LXC en Proxmox

A la hora de crear el contenedor LXC tienes varias opciones y la que voy a tocar yo es la de crear un contenedor con Debian 12.
Si prefieres otra distribución, puedes elegir la que más te guste pero ten en cuenta que los comandos y configuraciones pueden variar.

Respecto a los recursos, yo he asignado 1 CPU, 512MB de RAM y 8GB de almacenamiento. Puedes ajustar estos valores según tus necesidades.

Recuerda que debes tener una IP fija en tu red local para poder acceder al contenedor.

Una vez creado el contenedor mantenlo apagado y sigue con el siguiente paso.

### 2. Configurar el contenedor LXC

Una vez creado el contenedor, antes de encederlo, vamos a configurar unas cuantas cosas.

Buscamos el archivo de configuración del contenedor dentro de `/etc/pve/nodes/NODE/lxc/CONTAINER_ID.conf` y añadimos las siguientes líneas:

```bash
lxc.cgroup.devices.allow: c 10:200 rwm
lxc.cgroup2.devices.allow: c 10:200 rwm
lxc.mount.entry: /dev/net dev/net none bind,create=dir
lxc.mount.entry: /dev/net/tun dev/net/tun none bind,create=file
```

Estas líneas permiten que el contenedor pueda usar el dispositivo `tun` que es necesario para OpenVPN.
Ahora vamos a configurar los permisos de `/dev/net/tun` mediante el siguiente comando:

```bash
chwon 100000:100000 /dev/net/tun
```

Con esto ya podemos encender el contenedor.

### 3. Instalar OpenVPN

Una vez encendido el contenedor, accedemos a él bien mediante noVNC o mediante SSH y ejecutamos los siguientes comandos:

```bash
apt-get update
apt-get upgrade -y
apt-get install openvpn git -y
git clone https://github.com/Nyr/openvpn-install
cd openvpn-install
bash ./openvpn-install.sh
```

Con estos comandos instalamos OpenVPN y ejecutamos el script de instalación que nos guiará en la configuración de OpenVPN.
La información que te pedira es la siguiente:

- Dirección IP pública del servidor
  - Pon tu IP pública o tu dominio si tienes uno. En caso de no contar con ninguno, podria ser interesante usar un DDNS como No-IP.
- Protocolo (UDP o TCP)
  - Si no sabes cual elegir, elige UDP, es más rápido.
- Puerto (recomendado 1194)
  - Si tienes dudas, deja el puerto por defecto aunque puede que tu ISP lo bloquee, en ese caso tendrás que cambiarlo.
- DNS
  - Preferencia personal, yo elegí Cloudflare (1.1.1.1)
- Nombre del cliente
  - Nombre del archivo de configuración que se generará, yo puse mi nombre.

Una vez terminado el script, se generará un archivo `.ovpn` que deberás descargar y guardar en un lugar seguro.
Este archivo es el que necesitarás para conectarte a tu servidor OpenVPN.

> Si vas a pasarlo entre ordenadores, te recomiendo hacerlo mediante SCP.
> Si vas a pasarlo a un dispositivo móvil, te recomiendo hacerlo mediante USB o no pasarlo en claro como minimo.
{: .prompt-warning }

### 4. Configurar el router

Para que OpenVPN funcione debemos permitir el tráfico de OpenVPN en nuestro router.

Para ello, debemos acceder al portal de configuración de nuestro router, normalmente mediante la dirección `192.168.1.1` aunque puede variar según el fabricante y la configuración de tu red.

Una vez dentro, debemos buscar la sección de reenvío de puertos o NAT y añadir una regla para el puerto que hemos elegido en el paso anterior.

![Router](/assets/img/posts/openvpn-router.png)
_Figura 1: Configuración del router_

Siento no poder ser más específico pero cada router es un mundo y no puedo abarcar todos los modelos y fabricantes.

### 5. Conectar a OpenVPN

Una vez configurado el router, ya podemos conectarnos a nuestro servidor OpenVPN.

Si estas en Windows, macOS o un dispositivo móvil, puedes usar el cliente oficial de OpenVPN.

Si estas en Linux puedes o bien usar la herramienta openvpn o bien configurar NetworkManager o en su defecto tu entorno gráfico para que se conecte automáticamente.

## Referencias

[^1]: [OpenVPN Data Channel Offload](https://openvpn.net/as-docs/openvpn-data-channel-offload.html)
[^2]: [OpenVPN install on Proxmox LXC](https://www.youtube.com/watch?v=nsy9acOKnPo)
