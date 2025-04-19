---
title: ¿Qué distribución de Linux elegir?
author: Xabierland
description: >-
  Descubre las diferencias entre las distribuciones de Linux más populares y elige la que mejor se adapte a tus necesidades.
date: 2024-11-09
categories: [Blog, Sistemas Operativos]
tags: [Linux, Distribuciones, Comparativa]
published: false
---

## Introducción

### ¿Que es Linux

### ¿Que es una distribución de Linux?

## Clasificación de las distribuciones de Linux

Existen unos puntos claves en los que debemos fijarnos a la hora de elegir una distribución de Linux y los voy a dividir en dos ordenes siendo el primero, al menos para mi, el más a tener en cuenta.

- Primer orden
  - Ciclo de lanzamiento
  - Gestor de paquetes
  - Init System
  - Mutabilidad
- Segundo orden
  - Biblioteca C
  - Entorno de escritorio o gestor de ventanas
  - Comunidad/Soporte

### Primer orden

#### Ciclo de lanzamiento

El ciclo de lanzamiento de una distribución de Linux es el tiempo que pasa entre una versión y otra.

Hay dos tipos de lanzamientos, los de tipo **Rolling Release** y los de tipo **Fixed Release**.

- **Rolling Release**: Son distribuciones que no tienen una versión concreta y se actualizan de forma continua. Las actualizaciones se van lanzando a medida que se van probando y no hay una versión concreta. Ejemplos de este tipo de distribuciones son Arch Linux, Manjaro, openSUSE Tumbleweed, etc.
  - **Ventajas**: Siempre tienes la última versión de los programas y del sistema operativo.
  - **Desventajas**: Pueden ser inestables y tener fallos de seguridad.
- **Fixed Release**: Son distribuciones que tienen una versión concreta y se actualizan de forma periódica. Las actualizaciones se lanzan en forma de versiones concretas y se actualizan a través de un sistema de paquetes. Ejemplos de este tipo de distribuciones son Debian, Ubuntu, Fedora, etc.
  - **Ventajas**: Son más estables y seguras.
  - **Desventajas**: No siempre tienes la última versión de los programas y del sistema operativo.

#### Gestor de paquetes


#### Init System

El Init System es el primer proceso que se ejecuta en el sistema operativo y es el encargado de iniciar el resto de procesos y servicios del sistema. Existen varios Init Systems pero los más populares son **SysVinit**, **Systemd** y **OpenRC**.

- **SysVinit**: Es el sistema de inicio más antiguo y simple. Se basa en scripts de inicio y es el más utilizado en sistemas Unix y Linux.
- **Systemd**: Es el sistema de inicio más moderno y complejo. Se basa en unidades de servicio y es el más utilizado en las distribuciones modernas.
- **OpenRC**: Es un sistema de inicio simple y moderno. Se basa en scripts de inicio y es el más utilizado en distribuciones como Gentoo y Artix.

Hay mucha controversia sobre todo con Systemd ya que es un sistema de inicio muy complejo hasta el punto de que no solo hace de sistema de inicio sino que también hace de gestor de sesiones, de red, de logs, de contenedores, de máquinas virtuales, etc. Esto ha hecho que muchos usuarios y desarrolladores se quejen de que Systemd no respeta la filosofía Unix de "Haz una cosa y hazla bien".

Yo personalmente prefiero OpenRC o incluso runit ya que son sistemas de inicio más simples y respetan más la filosofía Unix y KISS. No obstante, es inegable el peso de Systemd en el mundo y es probable que en algun momento tengas que lidiar con él.

#### Mutabilidad

En el mundo de las distribuciones de Linux hay dos tipos de distribuciones, las **Mutables** y las **Inmutables**.

- **Mutables**: Es el tipo de distribución más común y es aquella en las que los ficheros y carpetas del sistema operativo pueden ser modificadas por el usuario. Ejemplos de este tipo de distribuciones son Ubuntu, Fedora, openSUSE, etc.
  - **Ventaja:** Tienes un control total sobre el sistema operativo. Pudiendo crear, modificar y eliminar ficheros y carpetas a tu antojo.
  - **Desventaja:** Esta libertad puede llevar a fallos de seguridad y a romper la instalación del sistema operativo.
- **Inmutables**: Es un tipo de distribución en la que la raiz del sistema operativo se monta en modo de solo lectura y por tanto los archivos y carpetas del sistema operativo no pueden ser modificadas por el usuario. Ejemplos de este tipo de distribuciones son Fedora Silverblue, openSUSE MicroOS, etc.
  - **Ventaja:** Es más seguro y estable ya que no se pueden modificar los ficheros y carpetas del sistema operativo.
  - **Desventaja:** No puedes modificar los ficheros y carpetas del sistema operativo.

También mencionar que al no poder modificar los ficheros y carpetas tampoco podemos instalar programas de forma tradicional. Para instalar programas en una distribución inmutable se utilizan contenedores o Flatpaks. Esto hace que la instalación de programas sea más segura y estable pero también más pesada, lenta y con peor integración con el sistema operativo.

Yo soy partidario de usar distribuciones inmutables es los ordenadores de uso más personal y diario. Aquellos ordenadores en donde no tengamos necesidad de instalar programas complejos o que requieran de una gran integracion y configuracion por parte del sistema operativo. Si vamos a usar el ordenador para navegar por internet, ver películas, escuchar música, etc. una distribución inmutable es la mejor opción en mi opinión.

Para los servidores y ordenadores de uso más técnico y profesional prefiero usar distribuciones mutables ya que me dan más libertad y control sobre el sistema operativo claro está, siempre y cuando seamos conscientes de los que hacemos. Al final mayor libertad implica mayor responsabilidad.

### Segundo orden

#### Biblioteca C

La biblioteca C es una biblioteca de funciones que proporciona las funciones básicas de programación en C. Existen varias bibliotecas C pero las más populares son **glibc** y **musl**.

- **glibc**: Es la biblioteca C más popular y es la que se utiliza en la mayoría de las distribuciones de Linux. Es una biblioteca muy completa y potente pero también muy pesada y lenta.
- **musl**: Es una biblioteca C más ligera y rápida que glibc. Es la biblioteca C que se utiliza en distribuciones como Alpine Linux.

#### Entorno de escritorio o gestor de ventanas

Aquí ya entramos en terreno más subjetivo ya que cada usuario tiene sus preferencias y necesidades. Hay muchos entornos de escritorio y gestores de ventanas pero los más populares son:

- **GNOME**: Es un entorno de escritorio moderno y completo. Es el entorno de escritorio que se utiliza en distribuciones como Fedora, Debian, etc.
- **KDE Plasma**: Es un entorno de escritorio moderno y completo. Es el entorno de escritorio que se utiliza en distribuciones como openSUSE, Manjaro, etc.
- **XFCE**: Es un entorno de escritorio ligero y sencillo. Es el entorno de escritorio que se utiliza en distribuciones como Xubuntu, MX Linux, etc.
- **LXDE**: Es un entorno de escritorio muy ligero y sencillo. Es el entorno de escritorio que se utiliza en distribuciones como Lubuntu, Knoppix, etc.
- **LXQt**: Es un entorno de escritorio ligero y moderno. Es el entorno de escritorio que se utiliza en distribuciones como Lubuntu, Knoppix, etc.
- **i3**: Es un gestor de ventanas minimalista y muy configurable. Es el gestor de ventanas que se utiliza en distribuciones como Arch Linux, Manjaro i3, etc.
- **Sway**: Es un gestor de ventanas minimalista y muy configurable. Es el gestor de ventanas que se utiliza en distribuciones como Arch Linux, Fedora i3, etc.
- **bspwm**: Es un gestor de ventanas minimalista y muy configurable. Es el gestor de ventanas que se utiliza en distribuciones como Arch Linux, Manjaro i3, etc.
- **dwm**: Es un gestor de ventanas minimalista y muy configurable. Es el gestor de ventanas que se utiliza en distribuciones como Arch Linux, Manjaro i3, etc.
- **Hyprland**: