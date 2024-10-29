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

A continuación, se detallan los pasos para instalar Docker y Podman en Windows y Linux.

### Docker

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

#### Instalación en Windows

- Descarga el instalador de Podman desde el repositorio oficial: [Podman](https://github.com/containers/podman/releases)
- Ejecuta el instalador descargado.
- Sigue las instrucciones del instalador.
- Una vez finalizada la instalación, reinicia el sistema.
- Abre una terminal de PowerShell y prueba a ejecutar un contenedor con Podman:

```powershell
podman run hello-world
```

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

> [!NOTE]
> Ingress no está habilitado por defecto en Minikube. Para habilitar Ingress, ejecuta el siguiente comando: 
> minikube addons enable ingress.

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
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube_latest_amd64.deb
sudo dpkg -i minikube_latest_amd64.deb
```

- Inicia Minikube:

```bash
minikube start
```
