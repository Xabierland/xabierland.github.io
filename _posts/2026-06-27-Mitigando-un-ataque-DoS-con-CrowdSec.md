---
title: Cómo sobreviví a un ataque DoS con CrowdSec
author: Xabierland
description: >-
  La historia de cómo una de mis aplicaciones autoalojadas se caía cada pocos
  días, el ataque DoS que había detrás, y cómo monté CrowdSec como IPS para
  detectarlo y bloquearlo de forma automática. Con todos los comandos y la
  configuración para que puedas replicarlo.
date: 2026-06-27 10:00
categories: [Seguridad, Administración de sistemas]
tags: [CrowdSec, DoS, IPS, Apache, Self-hosting, Proxy inverso]
---

## Introducción

Tengo varias aplicaciones autoalojadas detrás de un **proxy inverso**, que es mi
único punto de entrada desde internet. Una de ellas, una aplicación web servida
por **Apache + PHP**, empezó a comportarse mal: **cada pocos días el contenedor
moría**. Los recursos se disparaban al 100%, el servicio dejaba de responder y,
al rato, el contenedor caía sin más.

Lo primero que vi al revisar el contenedor fue el código de salida: **`137`**.
Ese número es `128 + 9`, es decir, el proceso recibió un **SIGKILL**. En una
máquina con poca RAM eso casi siempre significa lo mismo: **OOM** (Out Of
Memory), el kernel mató el proceso por quedarse sin memoria.

En este post cuento el diagnóstico completo, cómo encontré al atacante, y cómo
monté **CrowdSec** para que esto no me vuelva a tumbar el servicio. La idea es
que sirva de guía: van todos los comandos y configuraciones.

> Aviso: este artículo describe medidas **defensivas** sobre infraestructura
> propia. El "ataque" controlado del final lo lancé contra mi propio servicio
> para validar la defensa.
{: .prompt-warning }

## El diagnóstico

### Apache prefork desbocado

Revisando los logs de Apache justo antes de cada caída, el patrón era siempre el
mismo:

```text
[mpm_prefork:info] AH00162: server seems busy, (you may need to increase
StartServers, or Min/MaxSpareServers), spawning N children, there are 1 idle,
and 200 total children
```

**200+ procesos hijo de Apache** de forma sostenida. Con el MPM **prefork**,
cada hijo carga su propio intérprete de PHP y todo el footprint de la
aplicación. La configuración por defecto era demasiado generosa para la RAM de
la máquina:

```apache
<IfModule mpm_prefork_module>
    MaxRequestWorkers      250
    MaxConnectionsPerChild   0
</IfModule>
```

Hagamos la cuenta: **250 workers × ~25-40 MB** por proceso ≈ **6-9 GB** de RAM
en el peor caso. La máquina no tenía ni de lejos esa memoria. Bastaba un pico de
concurrencia para que Apache escalara los hijos, la RAM y el swap se agotaran, y
el OOM matara el proceso. El `137` cuadraba perfecto.

### El gatillo: un flood

¿Pero qué generaba ese pico de concurrencia? El tráfico normal no llegaba ni de
lejos a 200 peticiones simultáneas. Mirando los logs del **proxy inverso** (que
es quien recibe el tráfico real de internet), encontré el patrón:

- Una sola fuente lanzó **cientos de peticiones `GET /` en pocos minutos**.
- La home de la aplicación pesa ~240 KB y se renderiza con PHP en cada petición.
- La mayoría de esas peticiones **ni siquiera completaban** (el backend ya
  estaba ahogado), apareciendo con estado `-` en el log.

Era un **DoS volumétrico** de libro: saturar un endpoint caro con peticiones
concurrentes hasta agotar los recursos. Y revisando logs antiguos, la misma red
de origen ya había golpeado semanas antes. De ahí el "cada pocos días".

## Recuperando la IP real del atacante

Aquí me topé con un problema clásico al estar **detrás de un proxy inverso**:
Apache registraba como cliente la **IP del proxy**, no la del visitante real.
Todo el tráfico externo aparecía con la misma IP interna. Imposible identificar
al atacante desde la aplicación.

La solución es `mod_remoteip`, que lee la IP real desde la cabecera
`X-Forwarded-For` que envía el proxy. En la configuración de Apache:

```apache
LoadModule remoteip_module modules/mod_remoteip.so

RemoteIPHeader X-Forwarded-For
RemoteIPInternalProxy <IP_DE_TU_PROXY_INVERSO>
```

> Si cargas el módulo desde un fichero de configuración nuevo, necesitas
> **reiniciar Apache por completo**, no un simple `graceful reload`: un módulo
> recién añadido no registra sus hooks con una recarga en caliente.
{: .prompt-tip }

A partir de ahí, los logs ya mostraban la IP real de cada visitante. Y los logs
del propio proxy inverso (que sí guardan el `X-Forwarded-For`) me dieron la red
exacta del atacante.

## Mitigación inmediata: acotar Apache

Antes de montar nada sofisticado, la **red de seguridad** evidente: que Apache
no pueda pedir más memoria de la que cabe en la máquina. Bajé el techo de
workers a un valor que la RAM sí aguanta y forcé el reciclado de procesos:

```apache
<IfModule mpm_prefork_module>
    StartServers             3
    MinSpareServers          2
    MaxSpareServers          6
    MaxRequestWorkers       16
    MaxConnectionsPerChild 500
</IfModule>
```

Con esto, ante una saturación Apache **encola** las peticiones en lugar de
reventar la RAM. El servicio se degrada, pero **no muere**. También bajé el
`memory_limit` de PHP para que un único script no pudiera comerse toda la
máquina.

Esto evita la caída, pero no para al atacante: sigue golpeando. Para eso
necesitaba algo que **detectara y bloqueara** comportamientos abusivos de forma
automática. Ahí entra CrowdSec.

## La cura: CrowdSec

[CrowdSec](https://www.crowdsec.net/) es un IPS (Intrusion Prevention System)
colaborativo y open source. La idea es sencilla y se separa en dos mitades:

1. **El motor (engine)** lee tus logs, detecta comportamientos maliciosos según
   *escenarios* y genera **decisiones** (bans).
2. **El bouncer** aplica esas decisiones donde tú quieras (en mi caso, en el
   firewall con `nftables`).

Y como bonus, al estar conectado a la red comunitaria, recibes una **blocklist
con decenas de miles de IPs** de mala reputación reportadas por toda la
comunidad, bloqueándolas **antes** de que te ataquen.

Lo instalé en la misma máquina que el proxy inverso, ya que es donde están los
logs con el tráfico real de internet.

### 1. Instalar el motor

El repositorio oficial se añade con un `curl`:

```bash
curl -s https://install.crowdsec.net | sh
apt install crowdsec
```

Comprueba que arranca:

```bash
cscli version
systemctl status crowdsec
```

### 2. Leer los logs del proxy inverso

CrowdSec necesita saber qué logs leer y cómo parsearlos. El formato de logs de
mi proxy inverso no es uno estándar, así que escribí un **parser propio**.

Primero, la fuente de datos en `/etc/crowdsec/acquis.d/proxy.yaml`:

```yaml
filenames:
  - /ruta/a/los/logs/de/tu/proxy/*.log
labels:
  type: proxy
```

Luego el parser en `/etc/crowdsec/parsers/s01-parse/proxy-logs.yaml`. **Adapta
el patrón `grok` al formato concreto de tu proxy inverso**; lo importante es que
al final rellenes los mismos campos `evt.Meta.*` que esperan los escenarios HTTP
del hub:

```yaml
filter: "evt.Parsed.program == 'proxy'"
onsuccess: next_stage
name: custom/proxy-logs
description: "Parsea los logs de acceso de mi proxy inverso"
pattern_syntax:
  NOTDQUOTE: '[^"]*'
nodes:
  - grok:
      # Ajusta este patrón a TU formato de log
      pattern: '\[%{HTTPDATE:time_local}\] %{NOTSPACE:cache_status} %{NOTSPACE:upstream_status} %{NOTSPACE:status} - %{NOTSPACE:verb} %{NOTSPACE:scheme} %{NOTSPACE:http_host} "%{DATA:request}" \[Client %{IPORHOST:remote_addr}\] %{GREEDYDATA}'
      apply_on: message
      statics:
        - meta: log_type
          value: http_access-log
        - target: evt.StrTime
          expression: evt.Parsed.time_local
statics:
  - meta: service
    value: http
  - meta: source_ip
    expression: "evt.Parsed.remote_addr"
  - meta: http_status
    expression: "evt.Parsed.status"
  - meta: http_path
    expression: "evt.Parsed.request"
  - meta: http_verb
    expression: "evt.Parsed.verb"
  - meta: http_user_agent
    expression: "evt.Parsed.http_user_agent"
  - meta: target_fqdn
    expression: "evt.Parsed.http_host"
```

> Puedes probar el parser sin esperar a que llegue tráfico real con
> `cscli explain --log '<UNA LÍNEA DE LOG>' --type proxy`. Te muestra qué
> parsers casan y en qué escenarios vierte el evento. Inestimable para depurar.
{: .prompt-tip }

### 3. Un escenario para detectar el flood

Los escenarios HTTP del hub detectan scanning, fuerza bruta y CVEs, pero **un
flood de `GET /` con respuestas 200 no dispara ninguno** (no es un "ataque" por
patrón, es por volumen). Así que escribí uno propio en
`/etc/crowdsec/scenarios/http-flood.yaml`:

```yaml
type: leaky
name: custom/http-flood
description: "Banea IPs que floodean peticiones HTTP (DoS volumétrico)"
filter: "evt.Meta.log_type == 'http_access-log'"
groupby: "evt.Meta.source_ip"
capacity: 60
leakspeed: "1s"
blackhole: 5m
labels:
  type: dos
  remediation: true
  service: http
```

Es un *leaky bucket*: cada IP tiene un cubo con capacidad 60 que se vacía a 1
evento por segundo. Si una IP envía peticiones más rápido de lo que se vacía, el
cubo desborda y se genera el ban. Un flood sostenido cae en cuestión de
segundos.

Instala las colecciones HTTP base y recarga:

```bash
cscli collections install crowdsecurity/base-http-scenarios crowdsecurity/http-cve
cscli collections install crowdsecurity/whitelist-good-actors
systemctl reload crowdsec
```

> `whitelist-good-actors` evita banear crawlers legítimos (Google, Bing...)
> aunque su comportamiento parezca agresivo. Muy recomendable.
{: .prompt-tip }

### 4. El bouncer: aplicar los bans en el firewall

El motor solo **decide**; quien **bloquea** es el bouncer. Para firewall:

```bash
apt install crowdsec-firewall-bouncer-nftables
```

Aquí hay un detalle **importante si usas Docker** (mi proxy inverso corre en un
contenedor). El bouncer, por defecto, inserta sus reglas en el hook `input` de
nftables, que solo ve el tráfico **destinado a la propia máquina**. Pero el
tráfico hacia un contenedor Docker pasa por el hook `forward`, no por `input`.

> Si tu servicio está en un contenedor y el bouncer solo engancha en `input`,
> **estará baneando IPs que nunca llegan a verse afectadas**. Las versiones
> recientes del bouncer de nftables ya crean reglas tanto en `input` como en
> `forward`, cubriendo el tráfico a contenedores. Verifícalo:
{: .prompt-warning }

```bash
nft list table ip crowdsec | grep -E 'chain|hook'
# Debes ver un chain con 'hook input' y otro con 'hook forward'
```

A los pocos segundos, el bouncer sincroniza la blocklist comunitaria. Puedes ver
cuántas IPs tienes ya bloqueadas:

```bash
cscli decisions list
nft list set ip crowdsec crowdsec-blacklists-CAPI | grep -c timeout
```

En mi caso, varias **decenas de miles de IPs** de mala reputación bloqueadas
desde el primer momento, sin haber sido atacado todavía.

### 5. (Opcional) El dashboard

CrowdSec ofrece una consola web gratuita (hasta cierto punto) donde ves alertas,
decisiones y métricas en tiempo real. Se enrola con una *key* que obtienes en su
panel:

```bash
cscli console enroll <TU_ENROLLMENT_KEY>
systemctl restart crowdsec
```

Tras enrolarlo, hay que **aceptar el engine** en la web. Si prefieres no depender
de su nube, todo es consultable por CLI (`cscli alerts list`,
`cscli decisions list`, `cscli metrics`) o con un **Grafana** apuntando a las
métricas Prometheus que el motor ya expone localmente.

## Probándolo de verdad

Una defensa que no has probado no es una defensa. Desde un servidor externo
lancé una ráfaga controlada contra mi propio servicio:

```bash
PATHS="/ /admin /.env /wp-login.php /.git/config /phpmyadmin /shell.php"
seq 1 200 | xargs -P 15 -I{} sh -c '
  p=$(echo "$0" | tr " " "\n" | sed -n "$(( (RANDOM % 7) + 1 ))p")
  curl -s -o /dev/null --max-time 6 "https://tu-dominio/${p}"
' "$PATHS"
```

Segundos después, en el servidor de CrowdSec:

```console
$ cscli alerts list
 ID | value             | reason                                     | decisions
 17 | Ip:<ATACANTE>     | crowdsecurity/http-admin-interface-probing | ban:1
 16 | Ip:<ATACANTE>     | crowdsecurity/http-backdoors-attempts      | ban:1
 15 | Ip:<ATACANTE>     | crowdsecurity/CVE-2017-9841                | ban:1
```

**Tres escenarios distintos** detectados, ban automático generado y la IP metida
en el firewall en cuestión de segundos, **sin que yo tocara nada**. Verificado:
desde ese servidor externo, la web pasó a dar *timeout* total. Bloqueo efectivo
de punta a punta.

El flujo completo queda así:

```text
logs del proxy → el motor detecta comportamiento → genera un ban (4h por defecto)
→ el bouncer lo aplica en nftables → el atacante deja de llegar
```

## Un aviso: no te auto-banees

Las IPs residenciales son **dinámicas** y se reciclan. Si tu IP de casa hereda la
de un abusador previo, podría estar en la blocklist comunitaria y **CrowdSec te
bloquearía tu propio servicio**. Si te pasa, la solución es una *allowlist*:

```bash
cscli allowlists create home
cscli allowlists add home <TU_IP> -e 720h   # con expiración, no "never"
```

> Pon **siempre expiración** en allowlists de IPs dinámicas. Una entrada
> permanente acabaría protegiendo a quien herede esa IP en el futuro.
{: .prompt-warning }

## Conclusión

Lo que empezó como "se me cae el servicio cada pocos días" terminó siendo una
combinación de dos cosas: una **configuración de Apache demasiado optimista**
para la RAM disponible, y un **DoS volumétrico** que la disparaba.

La defensa quedó en tres capas:

1. **CrowdSec** en el proxy inverso → detecta y bloquea comportamientos abusivos
   e IPs de mala reputación, de forma automática.
2. **Límite de workers en Apache** → red de seguridad: si algo pasa el filtro,
   el servicio encola en vez de morir.
3. **IP real en los logs** (`X-Forwarded-For`) → visibilidad para el forense.

Ninguna capa es perfecta sola, pero juntas convierten un servicio que se caía
solo en uno que **se defiende solo**. Y lo mejor: CrowdSec es gratis, se
autoaloja y no depende de ningún tercero para lo esencial.

Si autoalojas servicios expuestos a internet, montar un IPS en tu punto de
entrada es de las mejores inversiones de tiempo que puedes hacer.
