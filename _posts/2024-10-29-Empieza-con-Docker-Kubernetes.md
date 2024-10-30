---
title: Empieza a trabajar con Docker y Kubernetes
author: Xabierland
description: >-
    Aprende a trabajar con Docker y Kubernetes
date: 2024-10-29
categories: [Tutorial]
tags: [Docker, Kubernetes, Containers, Orchestration]
---

<!-- markdownlint-disable MD024 -->

## Introducción

A peticiones de la audiencia, he decidido escribir un tutorial sobre cómo empezar a trabajar con Docker y Kubernetes. En este tutorial, aprenderás a instalar Docker y Kubernetes en tu sistema operativo, a crear y ejecutar contenedores con Docker, y a orquestar contenedores con Kubernetes.

## Instalación de gestor de contenedores (Docker/Podman)

Docker y Podman son herramientas de contenedorización que permiten ejecutar aplicaciones y sus dependencias de manera aislada en un entorno llamado "contenedor". Aunque ambos tienen propósitos similares, existen diferencias en su implementación, gestión de contenedores y enfoques de seguridad.

Ambas utilizan el estandar OCI (Open Container Initiative) para la creación de contenedores, lo que permite que los contenedores creados con Docker puedan ser ejecutados con Podman y viceversa.

A continuación, se detallan los pasos para instalar Docker y Podman en Windows y Linux.

### Docker

Docker como ya he dicho es una herramienta de contenedorización que permite ejecutar aplicaciones y sus dependencias de manera aislada en un entorno llamado "contenedor".

Docker usa un daemon que se ejecuta en segundo plano para levantar estos contenedores lo que obliga a tener que ejecutar dichos contenedores como root. Esto puede ser un problema tanto de seguridad como de flexibilidad a la hora de ejecutar contenedores.

Ademas, Docker tiene un problema en el entorno empresarial y es que el uso de Docker Desktop en entornos empresariales requiere una licencia de pago.

#### Instalación en Windows

Para instalar Docker en Windows, sigue los siguientes pasos:

- Descarga el instalador de Docker Desktop desde la página oficial de Docker: [Docker Desktop](https://www.docker.com/products/docker-desktop)
- Ejecuta el instalador descargado.
- Sigue las instrucciones del instalador.
- Una vez finalizada la instalación, reinicia el sistema.
- Abre una terminal de PowerShell y prueba a ejecutar un contenedor con Docker:

```powershell
docker run hello-world
```

#### Instalación en Linux

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

### Podman

Podman al igual que Docker es una herramienta de contenedorización que permite ejecutar aplicaciones y sus dependencias de manera aislada en un entorno llamado "contenedor".

Podman en cambio, no requiere un daemon para ejecutar los contenedores, estos se ejecutan como un proceso normal del sistema, lo que permite una mayor flexibilidad junto a que ya no es necesario ejecutar los contenedores como root.

Ademas Podman es una herramienta de código abierto y no requiere una licencia de pago para su uso en entornos empresariales.

Esto ha hecho que Podmna este ganando popularidad a lo largo de los años y que cada vez mas empresas lo esten adoptando como su herramienta de contenedorización frente a Docker.

#### Instalación en Windows

- Descarga el instalador de Podman desde el repositorio oficial: [Podman](https://github.com/containers/podman/releases)
- Ejecuta el instalador descargado.
- Sigue las instrucciones del instalador.
- Una vez finalizada la instalación, reinicia el sistema.
- Abre una terminal de PowerShell y prueba a ejecutar un contenedor con Podman:

```powershell
podman run hello-world
```

Tambien puedes instalar Podman en Windows usando Podman Desktop, que es una versión de Podman para Windows que incluye una interfaz gráfica y herramientas adicionales.

Para instalar Podman Desktop, sigue los siguientes pasos:

- Descarga el instalador de Podman Desktop desde la página oficial de Podman: [Podman Desktop](https://podman-desktop.io/)
- Ejecuta el instalador descargado
- Sigue las instrucciones del instalador.
- Una vez finalizada la instalación, reinicia el sistema.
- Abre Podman Desktop y prueba a ejecutar un contenedor.

Este ultimo metodo tambien es posible unicamente ejecutando el siguiente comando en PowerShell:

```powershell
winget install -e --id RedHat.Podman-Desktop
```

> Ten en cuenta que Podman funciona mediante WSL2. En principio ambos metodos deberian de activar WSL2 si no lo tienes activado. Si esto no es así, ejecuta el siguiente comando en PowerShell: `wsl --install --no-distribution`

#### Instalación en Linux

- Instala Podman:

```bash
sudo apt-get install podman
```

- Verifica que Podman se ha instalado correctamente:

```bash
podman run hello-world
```

## Instalación de orquestador de contenedores (Kubernetes)

### Kubectl

#### Instalación en Windows

- Abre una terminal de PowerShell como administrador.
- Creamos un directorio para almacenar el binario de `kubectl`:

```bash
mkdir C:\kubectl
cd C:\kubectl
```

- Descargamos el binario de `kubectl`:

```bash
curl -LO https://dl.k8s.io/release/v1.31.0/bin/windows/amd64/kubectl.exe
```

- Añadimos el directorio al `PATH` del sistema:

```bash
$env:Path += ";C:\kubectl"
[Environment]::SetEnvironmentVariable("Path", $env:Path, [EnvironmentVariableTarget]::User)
```

- Puede ser necesario reiniciar la terminal para que los cambios surtan efecto.

- Verificamos que `kubectl` se ha instalado correctamente:

```bash
kubectl version --client
```


#### Instalación en Linux

- Descarga el binario de `kubectl`:

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```

- Instala el binario de `kubectl`:

```bash
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

- Verifica que `kubectl` se ha instalado correctamente:

```bash
kubectl version --client
```

### Minikube

MiniKube es una herramienta que permite ejecutar un clúster de Kubernetes en un solo nodo. Es útil para probar aplicaciones y configuraciones de Kubernetes en un entorno local.

> Ingress no está habilitado por defecto en Minikube. Para habilitar Ingress, ejecuta el siguiente comando:
> `minikube addons enable ingress.`

#### Instalación en Windows

- Descarga el instalador o el binario de Minikube desde el repositorio oficial: [Minikube](https://github.com/kubernetes/minikube/releases)
- Crear un directorio para Minikube y mueve el binario descargado a ese directorio:

```bash
mkdir C:\minikube
cd C:\minikube
```

- Añade el directorio de Minikube al `PATH` del sistema:

```bash
$env:Path += ";C:\minikube"
[Environment]::SetEnvironmentVariable("Path", $env:Path, [EnvironmentVariableTarget]::User)
```

- Ejecuta el siguiente comando para iniciar Minikube:

```bash
minikube start
```

#### Instalación en Linux

- Descarga el binario de Minikube:

```bash
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 \
  && chmod +x minikube
```

- Lo mueves a un directorio que esté en tu `PATH`:

```bash
sudo install minikube /usr/local/bin/
```

- Inicia Minikube:

```bash
minikube start
```

## Referencias

- [Docker for Windows](https://docs.docker.com/docker-for-windows/install/)
- [Docker for Ubuntu](https://docs.docker.com/engine/install/ubuntu/)
- [Docker for Debian](https://docs.docker.com/engine/install/debian/)
- [Podman for Windows](https://podman.io/docs/installation#installing-on-mac--windows)
- [Podman Desktop for Windows](https://podman-desktop.io/docs/installation/windows-install)
- [Kubectl for Windows](https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/)
- [Kubectl for Linux](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)
- [Minikube](https://k8s-docs.netlify.app/en/docs/tasks/tools/install-minikube/)
