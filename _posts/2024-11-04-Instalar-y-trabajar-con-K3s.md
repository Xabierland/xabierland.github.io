---
title: Instalar y trabajar con K3s
author: Xabierland
description: >-
    Aprende a instalar y trabajar con K3s
date: 2024-11-04 10:04
categories: [Administración de sistemas, Orquestación de contenedores]
tags: [Kubernetes, Containers]
---

## Introducción

Hoy en día, Kubernetes es una de las herramientas de orquestación de contenedores más populares y utilizadas en el mundo de la informática y la programación. En este tutorial, aprenderás qué es K3s, por qué es tan popular y cómo instalarlo y trabajar con él.

### ¿Qué es K3s?

K3s es una distribución ligera de Kubernetes diseñada para entornos de desarrollo, pruebas y producción. K3s está optimizado para ejecutarse en máquinas con recursos limitados, como portátiles, servidores de borde y dispositivos IoT. No obstante, a diferencia de Minikube, K3s es adecuado para entornos de producción y puede escalar para admitir clústeres de Kubernetes de tamaño pequeño a mediano.

### ¿Por qué K3s?

K3s tiene varias ventajas sobre otras distribuciones de Kubernetes como son:

- Ligereza y eficiencia
  - K3s es una distribución ligera de Kubernetes que está diseñada para ser fácil de instalar y ejecutar en entornos con recursos limitados.
- Fácil instalación y configuración
  - K3s se puede instalar con un solo comando y se configura automáticamente para funcionar en la mayoría de los entornos.
- Compatibilidad con ARM
  - K3s es compatible con arquitecturas ARM, lo que lo hace ideal para dispositivos IoT y servidores de borde.
- Gestión simplificada
  - K3s simplifica la gestión de clústeres de Kubernetes al eliminar la necesidad de configuraciones complejas y herramientas adicionales ademas de permitir integración con herramientas de gestión de clústeres como Rancher.
- Actualizaciones rápidas y soporte para la comunidad
  - K3s se actualiza rápidamente y cuenta con una comunidad activa que proporciona soporte y contribuciones a la distribución.

K3s también tiene algunas desventajas:

- Limitaciones en entornos de producción grandes
  - K3s puede ser usado en entornos de producción, pero no esta diseñado para clústeres de gran escala.
- Soporte y personalización
  - K3s gana ligereza eliminando algunas características de Kubernetes, lo que puede limitar la personalización o el uso de herramientas específicas.
- Menor comunidad y soporte empresarial
  - Aunque K3s tiene una comunidad activa, esta, no tiene gran tamaño ni soporte empresarial.
- Limitaciones en la persistencia de datos
  - Al usar SQLite en lugar de etcd para almacenar los datos del clúster de Kubernetes, K3s puede tener limitaciones en la persistencia de datos en entornos de producción.

## Instalación y configuración de K3s

Para instalar K3s en Linux, sigue los siguientes pasos:

```bash
curl -sfL https://get.k3s.io | sh -
```

Para añadir workers al clúster de K3s, ejecuta el siguiente comando en el nodo worker:

```bash
curl -sfL https://get.k3s.io | K3S_URL=https://<IP>:6443 K3S_TOKEN=<TOKEN> sh -
```

Recuerda reemplazar `<IP>` y `<TOKEN>` con la dirección IP y el token del nodo maestro.

Para obtener el `<TOKEN>` ejecuta en el nodo maestro:

```bash
sudo cat /var/lib/rancher/k3s/server/node-token
```

## Como trabajar con K3s

Una vez que hayas instalado K3s, puedes empezar a trabajar con él utilizando la herramienta de línea de comandos `kubectl`.
A continuación, se muestran algunos comandos básicos para trabajar con K3s:

- `sudo k3s kubectl get nodes`: Muestra los nodos del clúster de K3s.
- `sudo k3s kubectl get pods`: Muestra los pods del clúster de K3s.
- `sudo k3s kubectl get services`: Muestra los servicios del clúster de K3s.
- `sudo k3s kubectl apply -f <archivo.yaml>`: Aplica la configuración de un archivo YAML en el clúster de K3s.
- `sudo k3s kubectl delete -f <archivo.yaml>`: Elimina la configuración de un archivo YAML del clúster de K3s.
- `sudo k3s kubectl exec -it <pod> -- <comando>`: Ejecuta un comando en un pod del clúster de K3s.
- `sudo k3s kubectl logs <pod>`: Muestra los logs de un pod del clúster de K3s.
- `sudo k3s kubectl port-forward <pod> <puerto_local>:<puerto_remoto>`: Reenvía un puerto de un pod del clúster de K3s al puerto local.
- `sudo k3s kubectl describe <recurso> <nombre>`: Muestra información detallada sobre un recurso del clúster de K3s.
- `sudo k3s kubectl edit <recurso> <nombre>`: Edita la configuración de un recurso del clúster de K3s.
- `sudo k3s kubectl scale <recurso> <nombre> --replicas=<número>`: Escala un recurso del clúster de K3s a un número específico de réplicas.
- `sudo k3s kubectl rollout restart <recurso> <nombre>`: Reinicia un recurso del clúster de K3s.

Es posible crear un alias para el comando `sudo k3s kubectl` para facilitar su uso. Para ello, añade la siguiente línea al archivo `.bashrc`:

```bash
echo "alias kubectl='sudo k3s kubectl'" >> ~/.bashrc
source ~/.bashrc
```

Estos son solo algunos de los comandos básicos que puedes utilizar para trabajar con K3s. Para obtener más información sobre cómo trabajar con K3s, consulta la [documentación oficial de Kubernetes](https://kubernetes.io/docs/reference/kubectl/).

## Referencias

- [Quick-Start Guide](https://docs.k3s.io/quick-start)
