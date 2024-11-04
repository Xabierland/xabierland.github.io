---
title: Editor LaTeX Online
author: Xabierland
description: >-
  Usa LaTeX en tu propia instancia de Overleaf/ShareLaTeX.
date: 2024-04-22
categories: [Tutorial, HomeLab]
tags: [LaTeX, Documentación, Escritura, Overleaf, ShareLaTeX]
---
<!-- markdownlint-disable MD029 -->
## ¿Que es LaTeX?

LaTeX es un sistema de composición de textos que se utiliza para la creación de documentos científicos y técnicos. LaTeX es muy popular en el ámbito académico y científico debido a su capacidad para producir documentos de alta calidad con un formato profesional.

## ¿Por qué usar LaTeX?

LaTeX tiene varias ventajas sobre otros sistemas de procesamiento de texto como Microsoft Word o Google Docs. Algunas de estas ventajas incluyen:

**Calidad tipográfica**: LaTeX produce documentos con una calidad tipográfica superior a la de otros sistemas de procesamiento de texto. LaTeX utiliza algoritmos avanzados para ajustar el espaciado, la justificación y la colocación de las letras y los párrafos, lo que resulta en un documento más legible y atractivo.

**Formato profesional**: LaTeX permite crear documentos con un formato profesional y consistente. LaTeX proporciona una amplia variedad de estilos y plantillas que permiten personalizar el aspecto de los documentos de acuerdo a las necesidades del autor.

**Facilidad de uso**: Aunque LaTeX tiene una curva de aprendizaje empinada, una vez que se domina, es muy fácil de usar. LaTeX utiliza un sistema de marcado sencillo que permite a los autores centrarse en el contenido del documento en lugar de en su formato.

**Compatibilidad**: LaTeX es un estándar de la industria en el ámbito académico y científico, por lo que es compatible con la mayoría de los editores y sistemas de publicación. LaTeX también es compatible con otros sistemas de procesamiento de texto como Microsoft Word y Google Docs, lo que facilita la colaboración y el intercambio de documentos.

**Software libre y de código abierto**: LaTeX es un software libre y de código abierto, lo que significa que es gratuito y que su código fuente está disponible para su modificación y distribución. LaTeX está disponible para la mayoría de los sistemas operativos, incluyendo Windows, macOS y Linux.

## ¿Cómo instalar LaTeX en local?

En Linux, LaTeX se instala a través de los paquetes de la distribución. En Debian, por ejemplo, se instala con el siguiente comando:

```bash
sudo apt install texlive-full
```

Y para compilar un documento LaTeX, se utiliza el comando `pdflatex`:

```bash
pdflatex documento.tex
```

Tambien puedes instalar en VSCode la extensión LaTeX Workshop que te permite compilar y visualizar el documento LaTeX en tiempo real.

## No quiero instalar LaTeX en local, ¿hay alguna alternativa?

Si no quieres instalar LaTeX en tu ordenador tienes dos opciones:

Usar un editor de LaTeX online como [Overleaf](https://www.overleaf.com/).

Montarte un servidor de LaTeX en tu VPS o Home Server

## ¿Cómo puedo montar un servidor de LaTeX?

Los pasos para montar un servidor de LaTeX son sencillos, a continuación te voy a ir explicando como lo he hecho yo en mi Home Server usando Proxmox y un contenedor LXC.

1 Crear un contenedor LXC en Proxmox con Debian (o la distribución que prefieras).

2 Actualiza el sistema y instala los paquetes necesarios:

```bash
apt update
apt dist-upgrade
apt install git curl ca-certificates
install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
chmod a+r /etc/apt/keyrings/docker.asc
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  tee /etc/apt/sources.list.d/docker.list > /dev/null
apt update
apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

3 Descargamos el servidor web de LaTeX:

```bash
git clone https://github.com/overleaf/toolkit.git ./overleaf-toolkit
cd overleaf-toolkit
```

4 Configuramos el servidor web de LaTeX:

```bash
bin/init
sed -i 's/127.0.0.1/0.0.0.0/g' config/overleaf.rc
sed -i 's/OVERLEAF_/SHARELATEX_/g' config/variables.env # No se porque pero el script de init no cambia las variables de entorno
```

5 Iniciamos el servidor web de LaTeX con `bin/up`.

6 Accedemos a la dirección IP de nuestro contenedor LXC mediante http://$IP/launchpad y creamos la cuenta de administrador.

7 Paramos el servidor web de LaTeX con `CTRL+C`.

8 De ahora en adelante, lo levantamos con `bin/start` y lo paramos con `bin/stop`.

9 Levantamos el servidor web de LaTeX y ejecutamos `bin/upgrade` para actualizar el servidor web de LaTeX.

10 Ejecutamos `bin/shell`para acceder al contenedor docker de LaTeX y ejecutamos los siguientes comandos para actualizar la instalación de LaTeX:

```bash
wget https://mirror.ctan.org/systems/texlive/tlnet/update-tlmgr-latest.sh
chmod +x update-tlmgr-latest.sh
./update-tlmgr-latest.sh
tlmgr option repository ctan
tlmgr update --self --all
tlmgr install scheme-full
```

11 Salimos del contenedor docker de LaTeX con `exit`.

12 Guardamos los cambios del contenedor docker de LaTeX:

```bash
docker commit sharelatex sharelatex/sharelatex:with-texlive-full
nano lib/docker-compose.base.yml
# Cambiamos la imagen a sharelatex/sharelatex:with-texlive-full
```

13 Levantamos el servidor con los cambios:

```bash
bin/stop
bin/docker-compose rm -f sharelatex
bin/up
```

14 Accedemos a la dirección IP de nuestro contenedor LXC mediante http://$IP/login y ya podemos empezar a usar LaTeX.
