---
title: Monta un servidor con Proxmox
author: Xabierland
description: >-
  Monta un servidor con Proxmox en tu casa para tener control sobre tus servicios y aplicaciones.
date: 2024-01-09
categories: [Tutorial, HomeLab]
tags: [Proxmox, Virtualización, Servidor, Redes, Seguridad]
---

## Introducción

Proxmox es una plataforma de virtualización de código abierto que combina dos tecnologías de virtualización: KVM (Kernel-based Virtual Machine) y LXC (Linux Containers). Proxmox es una solución ideal para montar un servidor en casa y tener control sobre tus servicios y aplicaciones.

KVM es una tecnología de virtualización completa que permite ejecutar máquinas virtuales con sistemas operativos completos. KVM utiliza la virtualización basada en hardware mediante QEMU para proporcionar un rendimiento óptimo y un aislamiento completo entre las máquinas virtuales.

LXC es una tecnología de contenedores que permite ejecutar aplicaciones en entornos aislados. Los contenedores son más ligeros que las máquinas virtuales y comparten el mismo kernel del sistema operativo host. Esto hace que los contenedores sean más rápidos y eficientes que las máquinas virtuales pero pueden tener limitaciones en cuanto a la seguridad y el aislamiento.

Proxmox combina lo mejor de ambos mundos: máquinas virtuales para aplicaciones que requieren un aislamiento completo y contenedores para aplicaciones que requieren un rendimiento óptimo.

## Instalación de Proxmox

### 1. Pre-requisitos

Para instalar Proxmox en tu casa necesitas un servidor con al menos 4GB de RAM, 50GB de almacenamiento y un procesador compatible con la virtualización por hardware (Intel VT-x o AMD-V).

Estos son los requisitos mínimos para instalar Proxmox, pero te recomiendo que tengas al menos 8GB de RAM y 512GB de almacenamiento para poder al menos correr de cuatro a seis servicios sin problemas.

Además, necesitas una conexión a Internet para descargar la imagen ISO de Proxmox y una dirección IP estática para acceder al servidor desde la red local.

### 2. Crea un medio de instalación

Descarga la última versión de Proxmox VE desde la [página oficial](https://www.proxmox.com/es/proxmox-ve) y graba la imagen ISO en un USB o DVD.

Si estas en un sistema Windows o macOS, puedes usar [Rufus](https://rufus.ie/es/) o [Etcher](https://www.balena.io/etcher/) para grabar la imagen ISO en un USB.

Si estas en un sistema Linux, puedes usar `dd` para grabar la imagen ISO en un USB:

```bash
sudo dd if=proxmox-ve_7.1-1.iso of=/dev/sdX bs=4M status=progress oflag=sync
```

### 3. Instala Proxmox

Arranca el servidor desde el USB o DVD con la imagen de Proxmox y sigue las instrucciones del instalador. Durante la instalación, se te pedirá que introduzcas: un nombre para el nodo, una dirección IP estática para el servidor, una máscara de red, una puerta de enlace y un servidor DNS.

> Si no sabes bien como configurar estos valores te recomiendo que aprendas un poco sobre redes antes de seguir con la guía.
{: .prompt-warning }

Una vez finalizada la instalación, puedes acceder a la interfaz web de Proxmox desde un navegador web introduciendo la dirección IP del servidor en la barra de direcciones.

```bash
firefox https://IP_DEL_SERVIDOR:8006
```

### 4. Configura Proxmox

Existen multitud de configuraciones que puedes hacer en Proxmox voy a ir tocando algunas y en el futuro actualizaré la guía con más configuraciones a medida que las vaya aprendiendo.

### 5. Crea una máquina virtual

Lo primero que vamos ha hacer es elegir el sistema operativo que vamos a instalar en la máquina virtual. En mi caso, voy a instalar Debian 12.
Primero, descarga la imagen ISO de Debian 12 desde la [página oficial](https://www.debian.org/distrib/netinst) y guárdala en tu ordenador.
Accede a la interfaz web de Proxmox y selecciona el almacenamiento local. En el apartado `ISO Images` haz clic en `Upload` y selecciona la imagen ISO de Debian 12 que has descargado. También puedes pasar la URL de la imagen ISO para que el servidor la descargue automáticamente meidante el botón `Download from URL`.

Una vez tengamos la imagen ISO en el almacenamiento local, vamos a crear una nueva máquina virtual. Haz clic en `Create VM` y sigue los pasos del asistente para configurar la máquina virtual.

Aquí, dependiendo de tus necesidades y recursos, puedes ajustar la cantidad de CPU, RAM y almacenamiento que quieres asignar a la máquina virtual.

Una vez creada la máquina virtual, arráncala y sigue las instrucciones del instalador de Debian 12 como si estuvieras instalando Debian en un ordenador físico.

### 6. Crea un contenedor LXC

Para crear un contenedor LXC en Proxmox lo primero que necesitas es una plantilla. Proxmox viene con varias plantillas preinstaladas que puedes usar para crear contenedores LXC. Yo, te recomiendo tambien activar las plantillas de TurnKey Linux[^1] que son muy útiles ya que vienen con aplicaciones preinstaladas. Para activar las plantillas de TurnKey Linux en caso de que no esten ya, entra en la consola del nodo y ejecuta el siguiente comando:

```bash
pveam update
```

Ahora, puedes crear un contenedor LXC desde la interfaz web de Proxmox. Haz clic en `Create CT` y selecciona la plantilla que quieres usar para el contenedor. Ajusta los recursos según tus necesidades y haz clic en `Create` para crear el contenedor.

Algunos servicios pueden requerir de configuraciones adicionales no disponibles desde la interfaz web, en estos casos, tendras que acceder al nodo mediante SSH o desde la consola de Proxmox y configurar el contenedor manualmente.

## Referencias

[^1]: [TurnKey Linux](https://www.turnkeylinux.org/)
