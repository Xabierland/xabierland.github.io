---
layout: blog
title: Configuración de FreeBSD
author: Xabier Gabiña Barañano
date: 2024-04-05
view: false
---

## ¿Que es FreeBSD?

FreeBSD es un sistema operativo basado en Unix que se deriva de BSD, la versión de Unix desarrollada en la Universidad de California, Berkeley. FreeBSD es un sistema operativo de código abierto y gratuito que se centra en la estabilidad, el rendimiento y la seguridad mediante la sencilles y la eficiencia.

FreeBSD tiene varias características que lo hacen único en comparación con otros sistemas operativos. Algunas de estas características incluyen:

### OpenZFS

OpenZFS es una plataforma de almacenamiento de código abierto que incluye la funcionalidad tanto de sistemas de archivos tradicionales como de gestor de volúmenes. Tiene muchas características avanzadas, que incluyen:

Protección contra la corrupción de datos. Verificación de integridad tanto para datos como para metadatos.

Verificación continua de la integridad y reparación automática "auto-reparadora".

Redundancia de datos con reflejo, RAID-Z1/2/3 [y DRAID].

Soporte para capacidades de almacenamiento altas, hasta 256 trillones de yobibytes (2^128 bytes).

Ahorro de espacio con compresión transparente utilizando LZ4, GZIP o ZSTD.

Encriptación nativa acelerada por hardware.

Almacenamiento eficiente con instantáneas y clones de escritura por copia.

Replicación local o remota eficiente: envía solo bloques modificados con ZFS send y receive.

Si te interesa, tienes el listado completo de características en la [página oficial de OpenZFS](https://openzfs.org/wiki/Features).

### Jails

Las Jails de FreeBSD son entornos de ejecución aislados que permiten ejecutar procesos de forma segura y limitada dentro de un sistema operativo FreeBSD. Estos entornos se basan en el concepto de chroot, que cambia el directorio raíz para un proceso dado, pero las Jails llevan este concepto más allá, proporcionando un aislamiento más completo del sistema de archivos, usuarios y recursos de red.

Cada Jail tiene su propio sistema de archivos raíz y su propio conjunto de usuarios, lo que garantiza que los procesos dentro de la Jail no puedan acceder ni afectar al sistema principal. Esto hace que las Jails sean útiles para la creación de entornos de desarrollo, pruebas o producción independientes y seguros dentro de un mismo sistema FreeBSD.

Esta idea es similar a la de los contenedores de Docker o Podman e incluso a las de las máquinas virtuales, pero con la ventaja de que las Jails son más ligeras, rápidas, mejor integradas con el sistema operativo y con mayor nivel de aislamiento lo que las hace más seguras.

### Ports

Los Ports de FreeBSD son una colección de scripts que automatizan la descarga, compilación e instalación de aplicaciones de terceros en un sistema FreeBSD. Cada Port contiene la información necesaria para compilar una aplicación específica, incluyendo las dependencias, opciones de configuración y parches necesarios.

La ventajas del uso de los ports frente a la aplicaciones que puedes encontrar en pkg es la misma que la de compilar un programa desde su código fuente frente a instalar un binario precompilado: mayor control sobre las opciones de compilación, mayor flexibilidad y la posibilidad de aplicar parches o personalizaciones específicas.

### Virtualización

FreeBSD incluye bhyve, un hipervisor que permite ejecutar máquinas virtuales en un sistema FreeBSD. bhyve es compatible con la virtualización de hardware y puede ejecutar sistemas operativos invitados como FreeBSD, Linux, Windows y otros sistemas operativos compatibles con x86.

A diferencia de las Jails, que unicamente pueden ejecutar aplicaciones del sistema FreeBSD mientras que bhyve permite ejecutar sistemas operativos completos, lo que lo hace ideal para la virtualización de servidores, pruebas de software y desarrollo de aplicaciones.

### Compatibilidad con Linux



### DTrace

### Capsicum

### Redes virtuales

### BSD License

## ¿Por qué FreeBSD?

## Instalación de FreeBSD

## Mi configuración de FreeBSD
