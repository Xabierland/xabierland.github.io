---
title: Instalar y trabajar con Docker
author: Xabierland
description: >-
    Aprende a instalar y trabajar con Docker
date: 2024-11-04 10:00
categories: [Administración de sistemas, Contenedores]
tags: [Docker, Containers]
---

## Introducción

Docker es una herramienta muy utilizada en el mundo de la informática y la programación. En este tutorial, aprenderás qué es Docker, por qué es tan popular y cómo instalarlo y trabajar con él.

### ¿Qué es Docker?

Docker es una herramienta de contenedorización que permite ejecutar aplicaciones y sus dependencias de manera aislada en un entorno llamado "contenedor". Esto es util para desarrolladores que quieren ejecutar aplicaciones en diferentes entornos sin tener que preocuparse por las dependencias de la aplicación. Docker también es util para administradores de sistemas que quieren ejecutar aplicaciones en servidores sin tener que preocuparse por las dependencias de la aplicación.

### ¿Por qué Docker?

Docker tiene varias ventajas sobre otras herramientas de contenedorización como son:

- Gran comunidad que tiene
  - Docker es bastante antiguo y tiene una gran comunidad que lo respalda con una gran cantidad de recursos y documentación.
- Facilidad de uso
  - Docker es fácil de instalar y usar lo que combinado con el punto anterior lo hace una herramienta muy accesible.
- Ecosistema integrado
  - Docker tiene un ecosistema integrado que incluye herramientas como Docker Compose, Docker Swarm y Docker Hub que facilitan la creación y gestión de contenedores.
- Compatibilidad y estabilidad.
  - La fama de Docker hace que se haya probado en una gran cantidad de entornos y sistemas operativos, lo que garantiza su compatibilidad y estabilidad.

No obstante, no todo son ventajas, Docker tiene tres problemas principales:

- Uso de daemon
  - Docker requiere un daemon que se ejecute en segundo plano para gestionar los contenedores, lo que puede consumir recursos y ralentizar el sistema ademas de suponer una brecha de seguridad al tener que ejecutarlo como root.
- Licencias
  - La version empresarial de Docker tiene una licencia propietaria que puede ser un problema para algunas empresas.
- Mayor consumo de recursos
  - Debido al uso de un daemon, Docker consume más recursos que otras herramientas de contenedorización como Podman o LXC.

## Instalación de Docker

Para instalar Docker en Linux, sigue los siguientes pasos:

- Añade el repositorio de Docker a Apt (Ubuntu)

```bash
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

- Añade el repositorio de Docker a Apt (Debian)

```bash
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

- Instala Docker:

```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

- Añade tu usuario al grupo `docker` para poder ejecutar Docker sin necesidad de ser `root`:

```bash
sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker
```

- Verifica que Docker se ha instalado correctamente:

```bash
docker run hello-world
```

### Instalacion de Windows y macOS

Si quieres instalar Docker en Windows o macOS, puedes descargar Docker Desktop desde la [página oficial de Docker](https://www.docker.com/products/docker-desktop).

## Crear y ejecutar contenedores con Docker

Para empezar un poco con Docker, vamos a ejecutar un contenedor de ejemplo. Para ello, vamos a ejecutar un contenedor de `nginx` que es un servidor web ligero.

```bash
docker run -d -p 8080:80 nginx
```

Este comando ejecuta un contenedor de `nginx` en segundo plano (`-d`) y mapea el puerto `80` del contenedor al puerto `8080` del host (`-p 8080:80`). Para verificar que el contenedor se ha ejecutado correctamente, puedes abrir un navegador web y navegar a `http://localhost:8080`.

Para ver los contenedores que se están ejecutando en tu sistema, puedes ejecutar el siguiente comando:

```bash
docker ps
```

Para parar un contenedor, puedes ejecutar el siguiente comando:

```bash
docker stop <container_id>
```

Para eliminar un contenedor, puedes ejecutar el siguiente comando:

```bash
docker rm <container_id>
```

Para comprobar las imágenes que tienes en tu sistema, puedes ejecutar el siguiente comando:

```bash
docker images
```

Para eliminar una imagen, puedes ejecutar el siguiente comando:

```bash
docker rmi <image_id>
```

Tambien es posible ejecutar un contenedor en modo interactivo, para ello puedes ejecutar el siguiente comando:

```bash
docker run -it --rm busybox sh
```

Este comando ejecuta un contenedor de `busybox` en modo interactivo (`-it`) y elimina el contenedor una vez que se detiene (`--rm`). Para salir del contenedor, puedes ejecutar el comando `exit`.

Para más información sobre cómo trabajar con Docker, puedes consultar la [documentación oficial de Docker](https://docs.docker.com/).

### Dockerfile

Para crear una imagen personalizada con Docker, puedes utilizar un archivo llamado `Dockerfile`. Un `Dockerfile` es un archivo de texto que contiene las instrucciones necesarias para construir una imagen de Docker. Por ejemplo, el siguiente `Dockerfile` crea una imagen de `nginx` con un archivo HTML personalizado:

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

Este `Dockerfile` contiene las siguientes instrucciones:

- `FROM`: especifica la imagen base que se utilizará para construir la imagen personalizada.
- `ENV`: define una variable de entorno que se utilizará en la imagen personalizada.
- `COPY`: copia un archivo del host al contenedor.
- `EXPOSE`: expone un puerto del contenedor al host.
- `CMD`: especifica el comando por defecto que se ejecutará cuando se inicie el contenedor.

Una vez que hayas creado el `Dockerfile`, puedes construir la imagen con el siguiente comando:

```bash
docker build -t my-nginx .
```

Para ejecutar un contenedor con la imagen personalizada, puedes utilizar el siguiente comando:

```bash
docker run -d -p 8080:80 my-nginx
```

Para más información sobre cómo trabajar con `Dockerfile`, puedes consultar la [documentación oficial de Docker](https://docs.docker.com/engine/reference/builder/).

### Docker Compose

Docker Compose es una herramienta que permite definir y ejecutar aplicaciones multi-contenedor con Docker. Para utilizar Docker Compose, necesitas crear un archivo llamado `docker-compose.yml` que contiene la configuración de tus contenedores. Por ejemplo, el siguiente `docker-compose.yml` define una aplicación con dos contenedores: uno para `mysql` y otro para `phpmyadmin`:

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

Para ejecutar la aplicación con Docker Compose, puedes utilizar el siguiente comando:

```bash
docker-compose up -d
```

Para más información sobre cómo trabajar con Docker Compose, puedes consultar la [documentación oficial de Docker Compose](https://docs.docker.com/compose/).

## Referencias

- [Install Docker Engine](https://docs.docker.com/engine/install/)
- [Dockerfile overview](https://docs.docker.com/build/concepts/dockerfile/)
- [Docker Compose overview](https://docs.docker.com/compose/)
