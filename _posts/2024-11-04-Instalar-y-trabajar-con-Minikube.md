---
title: Instalar y trabajar con Minikube
author: Xabierland
description: >-
    Aprende a instalar y trabajar con Minikube
date: 2024-11-04 10:05
categories: [Tutorial, Contenedores y Orquestación]
tags: [Kubernetes, Containers]
---

## Introducción

Hoy en día, Kubernetes es una de las herramientas de orquestación de contenedores más populares y utilizadas en el mundo de la informática y la programación. En este tutorial, aprenderás qué es Minikube, por qué es tan popular y cómo instalarlo y trabajar con él.

### ¿Qué es Minikube?

Minikube es una herramienta que permite ejecutar un clúster de Kubernetes en una máquina local. Esto es útil para desarrolladores que quieren probar y desarrollar aplicaciones en Kubernetes sin tener que configurar un clúster completo. Recuerda que Minikube está diseñado para entornos de desarrollo y pruebas, no para entornos de producción.

### ¿Por qué Minikube?

Minikube tiene varias ventajas sobre otras herramientas de orquestación de contenedores como son:

- Facilidad de uso
  - Minikube es fácil de instalar y usar, lo que lo hace ideal para desarrolladores que quieren probar Kubernetes en su máquina local.
- Portabilidad
  - Minikube se puede ejecutar en cualquier máquina, lo que lo hace ideal para entornos de desarrollo y pruebas.
- Compatibilidad con Kubernetes
  - Minikube es compatible con Kubernetes, lo que significa que puedes probar tus aplicaciones en un entorno similar al de producción.
- Integración con herramientas de desarrollo
  - Minikube se integra con herramientas de desarrollo como kubectl y Helm, lo que facilita el desarrollo y despliegue de aplicaciones en Kubernetes.

No obstante, Minikube también tiene algunas desventajas:

- Limitaciones de escala
  - Minikube está diseñado para entornos de desarrollo y pruebas, por lo que puede tener limitaciones de escala en comparación con un clúster de Kubernetes completo.
- Recursos
  - Minikube consume recursos de la máquina local, por lo que puede ralentizar el sistema si se ejecutan múltiples contenedores.

## Instalación de Minikube

### Linux

Para instalar Minikube en Linux, sigue los siguientes pasos:

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube && rm minikube-linux-amd64
```

### macOS

Para instalar Minikube en macOS, sigue los siguientes pasos:

```zsh
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-darwin-amd64
sudo install minikube-darwin-amd64 /usr/local/bin/minikube
```

### Windows

Para instalar Minikube en Windows, sigue los siguientes pasos:

> ¡Recuerda ejecutar powershell como administrador!

```powershell
New-Item -Path 'c:\' -Name 'minikube' -ItemType Directory -Force
Invoke-WebRequest -OutFile 'c:\minikube\minikube.exe' -Uri 'https://github.com/kubernetes/minikube/releases/latest/download/minikube-windows-amd64.exe' -UseBasicParsing
$oldPath = [Environment]::GetEnvironmentVariable('Path', [EnvironmentVariableTarget]::Machine)
if ($oldPath.Split(';') -inotcontains 'C:\minikube'){
  [Environment]::SetEnvironmentVariable('Path', $('{0};C:\minikube' -f $oldPath), [EnvironmentVariableTarget]::Machine)
}
```

## Crear y gestionar un clúster con Minikube

Para crear un clúster de Kubernetes con Minikube, puedes usar el siguiente comando:

```bash
minikube start
```

En caso de que quieras detener el clúster, puedes usar el siguiente comando:

```bash
minikube stop
```

Para eliminar el clúster, puedes usar el siguiente comando:

```bash
minikube delete
```

Para ver el estado del clúster, puedes usar el siguiente comando:

```bash
minikube status
```

Para acceder al dashboard de Kubernetes, puedes usar el siguiente comando:

```bash
minikube dashboard
```

Para desplegar una aplicación en el clúster, puedes usar el siguiente comando:

```bash
minikube kubectl create deployment hello-minikube --image=k8s.gcr.io/echoserver:1.4
```

Para exponer la aplicación, puedes usar el siguiente comando:

```bash
minikube kubectl expose deployment hello-minikube --type=NodePort --port=8080
```

Para acceder a la aplicación, puedes usar el siguiente comando:

```bash
minikube service hello-minikube
```

## Referencias

- [minikube start](https://minikube.sigs.k8s.io/docs/start/?arch=%2Flinux%2Fx86-64%2Fstable%2Fbinary+download)
- [handbook](https://minikube.sigs.k8s.io/docs/handbook/)
