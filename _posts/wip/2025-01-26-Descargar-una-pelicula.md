---
title: ¿Como de difícil puede ser descargar una película?
author: Xabierland
description: >-
  
date: 2025-01-26
categories: [Tutorial]
tags: [Películas]
published: false
---

Me he pasado el finde semana viendo las mejores peliculas de Maria Schell pero se me ha hecho imposible encontrar la Rose Bernd (1957) en ningun lado ni idioma en paginas "legitimas" como Netflix, HBO, Amazon, Prime Video, SkyShowTime, Disney+, Filmin, Movistar+ y Mubi. Tambien he buscado en todos los distribuidores de Torrents tanto mediante Stremio como mediante la web de RARBG y no he encontrado nada. No obstante no soy de los que se rinden facilmente y he decidido buscar la pelicula a mano. Despues de **horas** en DuckDuckGo he encontrado una pagina web llamada `streamkiste.tv`.

Aunque este podria haber sido el final de mi epopeya, la pelicula esta unicamente en aleman y no tiene subtitulos y como ni se aleman ni tengo la capacidad de aprenderlo en 5 minutos, he tenido que buscar una solucion para poder ver la pelicula. Lo mas sencillo que se me ha ocurrido ha sido descargar la pelicula y buscar los subtitulos en español pero esto no ha sido tan sencillo como podria parece.

La pagina web no ofrecia la posibilidad de descargar la pelicula por lo que ahora es donde he tenido que ponerme creativo. Lo primero he abierto la consola de desarrollador de Firefox y desde la herramienta de red he buscado la URL desde la que el elemento frame cargaba la pelicula.

![Link reproductor](/assets/img/posts/pelicula1.png)

Esto me ha llevado a una pagina web en la que se cargaba un reproductor de video. Obviamente la pelicula se estaba reproduciendo sobre HTTP mediante el protocolo HLS (HTTP Live Streaming). Este protocolo usa un archivo .m3u8 que contiene la lista de segmentos de video que se van a reproducir en formato TS (Transport Stream). Por lo que una vez más desde la consola de desarrollador he buscado la URL del archivo .m3u8.

![Archivo .m3u8](/assets/img/posts/pelicula2.png)

La pagina envia dos archivos .m3u8. Primero el master.m3u8 que contiene la lista de calidades y sus respectivos archivos index.m3u8. A nosotros nos interesa el archivo index.m3u8 que es la lista de segmentos de video.

Ahora todo lo que necesitamos es hacer click en el index.m3u8 y descargarlo. Una vez descargado usamos ffmepg para descargar los segmentos de video y unirlos en un solo archivo de video mediante el siguiente comando:

```bash
ffmpeg -i "URL_DEL_ARCHIVO_M3U8" -c copy -bsf:a aac_adtstoasc video.mp4
```

¿Facil no?

**ERROR**

Obviamente la pagina web no nos lo va a poner tan facil y usan un sufijo en todos los archivos tanto .m3u8 como .ts para evitar que los descarguemos.
En este punto solo se me ocurrian dos opciones, o carga la pelicula desde el navegador y una vez cargada descargaba a mano todos los archivos .ts o crear un script que descargara todos los archivos .ts.
Obviamente como buen informatico he optado por la segunda opcion.
