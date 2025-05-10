---
title: Inicia en Monero
description: >-
  Mi experiencia empezando en el mundo de Monero.
author: Xabierland
date: 2024-01-31
categories: [Blogs]
tags: [Monero, Criptomoneda, Privacidad]
---

## Introducción

En este post quiero hablar un poco de mi experiencia iniciándome en el mundo de Monero.  
Explicaré cómo he comenzado, qué he hecho, qué he aprendido, e iré actualizando el post con el tiempo, añadiendo los errores y nuevos conocimientos que vaya adquiriendo. De esta forma, espero que este post sirva de ayuda a otras personas que quieran empezar en el mundo de Monero y no sepan por dónde comenzar.

## ¿Qué es Monero?

Monero es una criptomoneda que se centra en la privacidad y el anonimato. Es un *fork* de Bytecoin, que a su vez es un *fork* de Bitcoin. Monero es una criptomoneda descentralizada, lo que significa que no hay una entidad central que controle la red. Esto la convierte en una criptomoneda resistente a la censura y a la confiscación. Además, Monero es de código abierto, lo que significa que cualquiera puede contribuir a su desarrollo.

## Inicio en Monero

### Crear una *wallet*

La forma correcta de generar una *wallet* es mediante una *paper wallet*. Una *paper wallet* es una *wallet* que se genera de forma *offline* y se imprime en papel. De esta forma, la *wallet* no está expuesta a Internet y, por tanto, no puede ser hackeada. La forma más segura de generar una *paper wallet* es mediante un sistema operativo en vivo, como Tails. Tails es un sistema operativo que se ejecuta desde un USB y que no deja rastro en el ordenador. De esta forma, si el ordenador está infectado con *malware*, este no podrá infectar a Tails.

Primero, hay que descargar el archivo `wallet-generator.zip` desde este [enlace](https://web.getmonero.org/generator/) o accediendo desde la web oficial y descomprimirlo en un USB. Luego, arrancamos Tails y abrimos el archivo `wallet-generator.html` que se encuentra en el USB para generar la *wallet*. Esto dará como resultado cuatro claves: pública, privada, de vista y de gasto.

- **Pública**: Es la clave que se comparte con otras personas para recibir pagos.
- **Privada (Mnemonic seed)**: Es la clave que se usa para restaurar la *wallet* en caso de pérdida.
- **De vista**: Es la clave que se usa para ver el saldo de la *wallet*.
- **De gasto**: Es la clave que se usa para gastar el saldo de la *wallet*.

Una vez generada, hay varias opciones para guardar la *wallet*, como USB, CD o papel. Como mínimo, debería guardarse en tres sitios diferentes y de forma segura.

### Obtener el software

En cuanto a software, hay varias opciones, pero recomiendo usar la aplicación oficial de Monero, de la cual tenemos dos variantes: Monero GUI y Monero CLI. La diferencia principal es que una es una interfaz gráfica y la otra es una interfaz de línea de comandos. Yo suelo usar Monero GUI, pero es cuestión de gustos.

Una vez obtenida la aplicación desde el sitio oficial de Monero y verificada la integridad del archivo, procedemos a descomprimirlo (al menos en el caso de Linux). Dentro de la carpeta encontraremos varios archivos, de los cuales nos interesan dos: `monero-wallet-gui` y `monerod`. El primero es la aplicación de Monero y el segundo es el demonio de Monero. El demonio es el encargado de sincronizar la *blockchain* de Monero y de mantenerla actualizada. La aplicación de Monero se conecta al demonio para poder funcionar. También es posible usar un demonio remoto, tanto propio como de terceros. Una de las opciones más recomendables y, por tanto, más seguras (aunque más laboriosas), es usar una VPS para alojar el demonio de Monero sobre Tor.

En caso de querer ver nuestras *wallets* de Monero en un dispositivo móvil, tenemos opciones como Monerujo y Cake Wallet.

### Cargar la *wallet*

Una vez generada la *wallet*, hay que cargarla en la aplicación de Monero. Para ello, abrimos la aplicación de Monero y seleccionamos la opción "Restaurar *wallet* desde claves o frase mnemónica" (*Restore wallet from keys or mnemonic seed*). Luego, introducimos las claves y ya tendremos la *wallet* cargada en la aplicación de Monero.  
También debemos especificar la dirección de la *blockchain* de Monero que vamos a usar. Si usamos el demonio de Monero, la dirección será `localhost`. Si usamos un demonio remoto, la dirección será la del demonio remoto. Si usamos un demonio remoto sobre Tor, la dirección será la dirección *onion* del demonio remoto. Según dónde hayas alojado el demonio, tendrás que cambiar la dirección.

### Adquirir Monero

Hay bastantes formas de adquirir Monero.

La más sencilla y segura es minar Monero. Minar Monero es el proceso de validar transacciones y añadir bloques a la *blockchain* de Monero. A cambio de este trabajo, se recibe una recompensa en Monero. Para minar Monero, es necesario tener un ordenador con una tarjeta gráfica potente y un software de minería. Existen varios programas de minería, pero recomiendo usar XMRig. XMRig es un software de minería de código abierto y muy fácil de usar. También es posible minar Monero en un *pool* de minería, que es un grupo de mineros que trabajan juntos para minar bloques y repartir la recompensa entre ellos. Minar en un *pool* de minería suele ser más rentable que minar en solitario.

La otra forma de adquirir Monero es comprándolo en un *exchange*. Un *exchange* es una plataforma que permite comprar y vender criptomonedas. Hay muchos *exchanges* para Monero. Te recomiendo que investigues y compares los diferentes *exchanges* antes de comprar Monero, ya que algunos son más seguros que otros. Aquí tienes la [lista de exchanges](https://www.getmonero.org/community/merchants/) que soportan Monero.

### Realizar compras

Una vez que tengas Monero en tu poder, puedes usarlo para comprar bienes y servicios como con cualquier otra moneda. No existen muchos comercios que acepten Monero, pero cada vez hay más. Una opción bastante conocida es [Anon Shop](https://anonshop.app/), pero solo funciona en Estados Unidos. Como suele ser habitual, tendrás que investigar y comparar para encontrar el comercio que mejor se adapte a tus necesidades.

## Conclusiones

Esta es una guía **muy** básica para empezar en el mundo de Monero. Hay muchas cosas que no he explicado, e incluso que desconozco. Recomiendo que leas la guía oficial de Monero, que dejaré en la bibliografía, y que busques en foros, canales de IRC o Reddit para aprender más sobre Monero.

## Bibliografía

- [Guía de Monero](https://www.getmonero.org/resources/user-guides/)
- [How to Run a Full Monero Node Over Tor](https://www.youtube.com/watch?v=nDBHhz00vjI)
