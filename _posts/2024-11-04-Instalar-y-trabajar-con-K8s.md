---
title: Instalar y trabajar con K8s bare-metal
author: Xabierland
description: >-
    Aprende a instalar y trabajar con K8s bare-metal 
date: 2024-11-04 10:03
categories: [Tutorial, Contenedores y Orquestación]
tags: [Kubernetes, Containers]
---

## Introducción

Hoy en día, Kubernetes es una de las herramientas de orquestación de contenedores más populares y utilizadas en el mundo de la informática y la programación.
En este tutorial aprenderemos a instalar y trabajar con Kubernetes (K8s) en un entorno bare-metal o on-premise.

### ¿Qué es Kubernetes (K8s)?

Kubernetes es un sistema de orquestación de contenedores de código abierto que automatiza la implementación, el escalado y la gestión de aplicaciones en contenedores. Diseñado por Google y lanzado como código abierto en 2014, Kubernetes se ha convertido en el sistema de orquestación de contenedores más popular y ampliamente utilizado.

Kubernetes bare-metal es la instalación de Kubernetes en un entorno on-premise o bare-metal, es decir, en servidores físicos en lugar de en la nube. Aunque Kubernetes es más conocido por su uso en la nube, también se puede instalar y utilizar en entornos bare-metal por varias razones

### ¿Por qué Kubernetes bare-metal?

Kubernetes bare-metal tiene varias ventajas sobre la implementación en la nube, como:

- Mayor rendimiento
  - Al eliminar la virtualización de la nube, se puede obtener un mejor rendimiento.
- Control total del entorno
  - Acceso directo al hardware y al sistema operativo lo que permite configuraciones avanzadas y adaptadas a las necesidades específicas.
- Costos reducidos
  - Al no tener proveedores de nube, se reducen los costos operativos aunque hay que tener en cuenta los costos de hardware y mantenimiento.
- Seguridad mejorada
  - Al tener control total del entorno, se pueden implementar medidas de seguridad avanzadas.
- Evitar el vendor lock-in
  - Al no depender de un proveedor de nube, es más facil migrar a otro proveedor o mantener el sistema on-premise.
- Cumplimiento normativo
  - Es más fácil cumplir con las regulaciones y normativas de seguridad al tener control total del entorno.

No obstante, Kubernetes bare-metal también tiene algunas desventajas, como:

- Mayor complejidad
  - La instalación y configuración de Kubernetes en bare-metal puede ser más compleja que en la nube.
- Menos automatización
  - Al no tener las herramientas de automatización de la nube, se requiere más trabajo manual.
- Menos escalabilidad
  - La escalabilidad en bare-metal puede ser más limitada que en la nube al necesitar más hardware físico.
- Soporte y mantenimiento
  - Al no tener un proveedor de nube, se requiere más soporte y mantenimiento interno.

## Instalación y configuración de Kubernetes bare-metal

### Requisitos previos

Para instalar Kubernetes en un entorno bare-metal, necesitarás los siguientes requisitos previos:

Instalar las herramientas necesarias

```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
sudo systemctl enable --now kubelet
```

Elegir un CRI (Container Runtime Interface):

- [X] containerd
- [ ] CRI-O
- [ ] cri-doclerd

> Ten en cuenta que la elección del CRI puede varias los pasos de instalación y configuración. Si vas a usar otro CRI, consulta la documentación [oficial de Kubernetes](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#installing-runtime)

Instalar containerd

```bash
sudo apt-get update && sudo apt-get install -y containerd
```

Configurar containerd

```bash
containerd config default | sudo tee /etc/containerd/config.toml > /dev/null 2>&1
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml
sudo systemctl restart containerd
```

> Este paso supone que estás usando systemd como init system. Si estás usando otro como openrc, upstart o sysvinit, consulta la documentación de containerd para configurarlo correctamente.

Desactivar el swap

```bash
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

Tener una IP estática

```bash
sudo bash -c 'cat <<EOF > /etc/network/interfaces.d/static-ip
auto eth0
iface eth0 inet static
    address 192.168.1.200
    netmask 255.255.255.0
    gateway 192.168.1.1
    dns-nameservers 8.8.8.8 8.8.4.4
EOF'

sudo systemctl restart networking
```

> Ten en cuenta que la dirección IP y la configuración de red pueden variar según tus necesidades.

Activar el IPv4 forwarding

```bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.ipv4.ip_forward = 1
EOF
sudo sysctl --system
```

### Crear el clúster maestro usando kubeadm

Para crear el clúster maestro, ejecuta el siguiente comando en el nodo maestro:

```bash
sudo kubeadm init --pod-network-cidr=192.168.0.0/16 --cri-socket=unix:///var/run/containerd/containerd.sock
```

> Ten en cuenta que el `--pod-network-cidr` es la dirección de la red de los pods y puede variar según tus necesidades.
> Si estás usando otro CRI, cambia el `--cri-socket` por el socket correspondiente.

Una vez que el comando se haya completado, se mostrará un mensaje con las instrucciones para unir los nodos al clúster.

Tambien deberás configurar el entorno de kubectl para que pueda comunicarse con el clúster:

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Ahora nos toca instalar un CNI (Container Network Interface) para que los pods puedan comunicarse entre sí.
Puede ver la lista de CNI soportados en la [aquí](https://github.com/containernetworking/cni).
Yo voy a instalar Calico:

```bash
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.29.0/manifests/tigera-operator.yaml
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.29.0/manifests/custom-resources.yaml
```

Con esto, ya deberíamos tener el nodo maestro configurado y listo para recibir trabajos. Ten en cuenta que este nodo no esta configurado para desplegar aplicaciones, solo para gestionar el clúster. Puedes hacer que este nodo también despliegue aplicaciones siguiendo [estos pasos](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#control-plane-node-isolation)

No obstante lo mejor es seguir con la creación de nodos trabajadores.

### Crear nodos trabajadores usando kubeadm

Una vez instaladas las dependencias del nodo de trabajo, puedes unirlos al clúster maestro.

Para añadir workers al clúster de K8s, ejecuta el siguiente comando en el nodo maestro:

```bash
sudo kubeadm join <IP>:6443 --token <TOKEN> --discovery-token-ca-cert-hash sha256:<HASH>
```

> Recuerda reemplazar `<IP>`, `<TOKEN>` y `<HASH>` con la dirección IP, el token y el hash del nodo maestro.
> Esta información se muestra al finalizar el comando `kubeadm init` en el nodo maestro.

## Empezar a trabajar con Kubernetes bare-metal

Una vez que hayas instalado Kubernetes en un entorno bare-metal, puedes empezar a trabajar con él utilizando la herramienta de línea de comandos `kubectl`.

A continuación, se muestran algunos comandos básicos para trabajar con Kubernetes:

- `kubectl get nodes`: Muestra los nodos del clúster de K8s.
- `kubectl get pods`: Muestra los pods del clúster de K8s.
- `kubectl get services`: Muestra los servicios del clúster de K8s.
- `kubectl apply -f <archivo.yaml>`: Aplica la configuración de un archivo YAML en el clúster de K8s.
- `kubectl delete -f <archivo.yaml>`: Elimina la configuración de un archivo YAML del clúster de K8s.
- `kubectl exec -it <pod> -- <comando>`: Ejecuta un comando en un pod del clúster de K8s.
- `kubectl logs <pod>`: Muestra los logs de un pod del clúster de K8s.
- `kubectl port-forward <pod> <puerto_local>:<puerto_remoto>`: Reenvía un puerto de un pod del clúster de K8s al puerto local.
- `kubectl describe <recurso> <nombre>`: Muestra información detallada sobre un recurso del clúster de K8s.
- `kubectl edit <recurso> <nombre>`: Edita la configuración de un recurso del clúster de K8s.
- `kubectl scale <recurso> <nombre> --replicas=<número>`: Escala un recurso del clúster de K8s a un número específico de réplicas.
- `kubectl rollout restart <recurso> <nombre>`: Reinicia un recurso del clúster de K8s.

Estos son solo algunos de los comandos básicos que puedes utilizar para trabajar con K8s. Para obtener más información sobre cómo trabajar con K8s, consulta la [documentación oficial de Kubernetes](https://kubernetes.io/docs/).

## Referencias

- [Bootstrapping clusters with kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/)