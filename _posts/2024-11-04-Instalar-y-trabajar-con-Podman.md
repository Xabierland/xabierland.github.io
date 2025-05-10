---
title: Instalar y trabajar con Podman
author: Xabierland
description: >-
    Aprende a instalar y trabajar con Podman
date: 2024-11-04 10:01
categories: [Administración de sistemas, Contenedores]
tags: [Podman, Containers]
---

## Introducción

Docker es una herramienta bastante utilizada en el mundo de la informática y la programación y que ha ido cogiendo fuerza con el paso de los tiempos. En este tutorial, aprenderás qué es Docker, por qué es tan popular y cómo instalarlo y trabajar con él.

### ¿Qué es Podman?

Podman es una herramienta de contenedorización que permite ejecutar aplicaciones y sus dependencias de manera aislada en un entorno llamado "contenedor". Esto es util para desarrolladores que quieren ejecutar aplicaciones en diferentes entornos sin tener que preocuparse por las dependencias de la aplicación. Podman también es util para administradores de sistemas que quieren ejecutar aplicaciones en servidores sin tener que preocuparse por las dependencias de la aplicación.

### ¿Por qué Podman?

Podman tiene varias ventajas sobre otras herramientas de contenedorización como son:

- Compatible con Docker
  - Podman es compatible con Docker, lo que significa que puedes usar imágenes de Docker en Podman sin problemas.
- Mayor seguridad
  - Podman es más seguro que Docker ya que no requiere ejecutar un daemon como root.
- Ligero y eficiente
  - Podman es más ligero y eficiente que Docker ya que no requiere un daemon en segundo plano.
- Licencia
  - Podman es de código abierto y no tiene una licencia propietaria como Docker.

Pero tiene algunas desventajas:

- Menor madurez
  - Podman es una herramienta más nueva que Docker y puede tener menos recursos y documentación.
- Ecosistema
  - Podman no tiene un ecosistema tan grande como Docker.
- Adopción y soporte
  - Podman no es tan popular como Docker y puede tener menos adopción, soporte y documentación.

## Instalación de Podman

Para instalar Podman en Linux, sigue los siguientes pasos:

```bash
sudo apt-get update
sudo apt-get install -y podman podman-compose
```

### Instalacion de Windows y macOS

Si quieres instalar Podman en Windows o macOS, puedes descargar Podman Desktop desde la [página oficial de Podman.io](https://podman.io/)

## Crear y ejecutar contenedores con Podman

Para empezar a trabajar con Podman, puedes crear un contenedor con el siguiente comando:

```bash
podman run -d -p 8080:80 nginx
```

Este comando crea un contenedor con el servidor web Nginx y lo ejecuta en el puerto 8080 de tu máquina. Puedes acceder al servidor web en tu navegador visitando `http://localhost:8080`.

Para ver los contenedores que están en ejecución, puedes usar el siguiente comando:

```bash
podman ps
```

Para detener un contenedor, puedes usar el siguiente comando:

```bash
podman stop <container_id>
```

Para eliminar un contenedor, puedes usar el siguiente comando:

```bash
podman rm <container_id>
```

Para comprobar las imágenes que tienes en tu sistema, puedes usar el siguiente comando:

```bash
podman images
```

Para eliminar una imagen, puedes usar el siguiente comando:

```bash
podman rmi <image_id>
```

También puedes ejecutar un contenedor en modo interactivo con el siguiente comando:

```bash
podman run -it --rm busybox sh
```

Este comando ejecuta un contenedor de BusyBox en modo interactivo y elimina el contenedor cuando se detiene. Para salir del contenedor, puedes ejecutar el comando `exit`.

Para más información sobre Podman, puedes consultar la [documentación oficial de Podman](https://docs.podman.io/en/latest/Commands.html).

### Podmanfile

Podman también soporta la creación de contenedores a través de un archivo llamado `Podmanfile`. Este archivo es similar a un `Dockerfile` y se utiliza para definir la configuración de un contenedor. Aquí tienes un ejemplo de un `Podmanfile`:

```Dockerfile
# Usamos como base la imagen oficial de nginx
FROM nginx:alpine

# Activamos el modo de mantenimiento
ENV MAINTENANCE_MODE=on

# Copiamos el archivo HTML personalizado al directorio de trabajo
COPY index.html /usr/share/nginx/html

# Exponemos el puerto 80
EXPOSE 80

# Comando por defecto
CMD ["nginx", "-g", "daemon off;"]
```

Este `Podmanfile` crea una imagen de Nginx con un archivo HTML personalizado y lo ejecuta en el puerto 80.

Una vez que hayas creado tu `Podmanfile`, puedes construir la imagen con el siguiente comando:

```bash
podman build -t my-nginx .
```

Para ejecutar un contenedor a partir de la imagen que has creado, puedes usar el siguiente comando:

```bash
podman run -d -p 8080:80 my-nginx
```

### Podman-compose

Podman también soporta la herramienta `podman-compose`, que es similar a `docker compose` y se utiliza para definir y ejecutar aplicaciones multi-contenedor. Aquí tienes un ejemplo de un archivo `podman-compose.yml

```yaml
version: '3'

services:
  db:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: example
      MYSQL_DATABASE: example
      MYSQL_USER: example
      MYSQL_PASSWORD: example
    volumes:
      - db_data:/var/lib/mysql

  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    ports:
      - "8080:80"
    environment:
      PMA_HOST: db
    depends_on:
      - db

volumes:
  db_data:
```

Para ejecutar la aplicación con `podman-compose`, puedes usar el siguiente comando:

```bash
podman-compose up -d
```

## Referencias

- [Podman Installation Instructions](https://podman.io/docs/installation)
- [Commands](https://docs.podman.io/en/latest/Commands.html)
