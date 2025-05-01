---
title: Tailscale para tu hogar
author: Xabierland
description: >-
  Aprende a instalar y configurar Tailscale en tu red doméstica para acceder a ella de forma segura, rápida y privada desde cualquier lugar.
date: 2025-05-01
categories: [Tutorial, HomeLab]
tags: [Tailscale, VPN, Redes, Seguridad, Proxmox]
---

## Introducción

Después de varios meses utilizando OpenVPN, he decidido migrar a Tailscale. La razón principal de este cambio es su simplicidad, facilidad de uso y rapidez en la configuración. Además, destaco su excelente integración con servicios modernos como Kubernetes, que estaré implementando próximamente.

Tailscale es una VPN de tipo "mesh" (malla) que permite conectar dispositivos de manera segura y privada a través de Internet. A diferencia de las VPN tradicionales como OpenVPN, Tailscale no requiere un servidor VPN centralizado, facilitando enormemente la gestión y la conexión de múltiples dispositivos.

Tailscale emplea el protocolo WireGuard, reconocido por su rendimiento excepcional, rapidez y robustez en seguridad y cifrado.

## Instalación y configuración en Proxmox

El primer paso consiste en crear un contenedor LXC en Proxmox. Personalmente, utilizo Debian, pero puedes elegir la distribución que prefieras.

Una vez creado el contenedor, es necesario modificar su configuración para permitir el acceso al dispositivo `tun`, requerido por Tailscale. Ejecuta los siguientes comandos en tu servidor Proxmox:

```bash
export CONTAINER_ID=100 # Reemplaza con el ID real de tu contenedor
echo "lxc.cgroup2.devices.allow: c 10:200 rwm" >> /etc/pve/lxc/$CONTAINER_ID.conf
echo "lxc.mount.entry: /dev/net/tun dev/net/tun none bind,create=file" >> /etc/pve/lxc/$CONTAINER_ID.conf
```

> Recuerda sustituir `CONTAINER_ID` por el ID correcto de tu contenedor.
{: .prompt-warning }

Ahora, inicia el contenedor y abre una terminal dentro del mismo para continuar con la instalación:

```bash
# Actualiza el sistema e instala dependencias necesarias
apt-get update && apt-get dist-upgrade -y && apt-get autoremove -y && apt-get install curl ethtool -y

# Habilita el reenvío de paquetes IP
sudo tee -a /etc/sysctl.d/99-tailscale.conf << EOF
net.ipv4.ip_forward = 1
net.ipv6.conf.all.forwarding = 1
EOF
sudo sysctl -p /etc/sysctl.d/99-tailscale.conf

# Habilitar el transport layer offload
NETDEV=$(ip -o route get 8.8.8.8 | cut -f 5 -d " ")
sudo ethtool -K $NETDEV rx-udp-gro-forwarding on rx-gro-list off


# Instala Tailscale
curl -fsSL https://tailscale.com/install.sh | sh
```

Para verificar que Tailscale se ha instalado correctamente, ejecuta:

```bash
systemctl status tailscaled
```

## Autenticación y configuración inicial

A continuación, inicia el servicio de Tailscale y autentica tu contenedor en tu cuenta de Tailscale:

```bash
tailscale up --advertise-routes=192.168.1.0/24
```

Este comando mostrará una URL que debes abrir en tu navegador para completar la autenticación del contenedor. Una vez autenticado, debes confirmar la ruta anunciada desde la [consola administrativa de Tailscale](https://login.tailscale.com/admin/machines).

![Tailscale Subnet](/assets/img/posts/tailscale-subnet.png)

> Tambien puedes configurar el servidor como `Exit node` para enrutar todo el tráfico a través de él hacia el internet mediante la flag `--advertise-exit-node`.
{: .prompt-tip }

Desde este punto, puedes seguir incorporando dispositivos adicionales a tu red instalando Tailscale en cada uno de ellos, permitiéndote acceder a tu red local desde cualquier parte del mundo de forma segura y sencilla.

Si utilizas un servidor DNS local, configura su dirección IP en la sección [DNS de Tailscale](https://login.tailscale.com/admin/dns). Esto te permitirá resolver los nombres de host de tu red local fácilmente.

![Tailscale DNS](/assets/img/posts/tailscale-dns.png)

## Conclusión

La adopción de Tailscale como solución VPN para entornos domésticos facilita enormemente el acceso seguro y privado a tu red local. Su simplicidad, rendimiento y facilidad de gestión son razones suficientes para recomendar su uso, especialmente si planeas integrar tecnologías modernas como Kubernetes.
