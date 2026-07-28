---
title: Cuando el kernel no distingue a tu administrador de un contenedor
author: Xabierland
description: >-
  Un `find -uid 1001` que devolvía ficheros que no eran de nadie me llevó a un
  agujero de conejo: por qué el UID de un contenedor es el mismo entero que el
  de tus usuarios, cómo elegir un rango que no colisione, las dos trampas de
  `usermod` que te dejan el sistema a medias, y cómo los user namespaces lo
  arreglan de verdad. Con las pruebas para que las repitas.
date: 2026-07-28 10:00
categories: [Seguridad, Administración de sistemas]
tags: [Linux, Kernel, Contenedores, Kubernetes, UID, User Namespaces, Ansible]
---

## Introducción

Estaba auditando qué había tocado una cuenta en una flota de servidores. Lo
típico: revisar accesos, comandos ejecutados y ficheros de su propiedad. Lancé un
`find` buscando lo que pertenecía a su UID, el 1001, y me devolvió cosas raras:
directorios de aplicaciones que esa persona no había visto en su vida, un binario
de un servicio del sistema y ficheros dentro de volúmenes de contenedores.

La explicación resultó ser bastante más interesante que el susto inicial: **esos
ficheros no eran suyos, pero para el kernel sí lo eran**. En una máquina que
corre contenedores sin *user namespaces*, un proceso dentro de un contenedor con
UID 1001 y una persona con UID 1001 son, literalmente, la misma identidad.

Este post es lo que aprendí arreglándolo: por qué pasa, cómo se elige un rango de
UID que no colisione, las dos trampas que te esperan al migrar cuentas ya
existentes, y por qué los *user namespaces* son la solución de verdad. Van todos
los comandos y las pruebas.

## El kernel no conoce nombres, solo números

Es fácil olvidarlo porque `ls -l` nos malacostumbra, pero **los ficheros no
pertenecen a usuarios, pertenecen a números**. El fichero `/etc/passwd` no es más
que una tabla de traducción que convierte esos números en nombres para que los
humanos podamos leerlos.

Compruébalo con `ls -n`, que muestra los números en crudo en vez de resolverlos:

```bash
ls -l /ruta   # muestra:  -rw-r--r-- 1 miusuario  miusuario  ...
ls -n /ruta   # muestra:  -rw-r--r-- 1 1001       1001       ...
```

Aquí está la clave: **cada contenedor tiene su propio `/etc/passwd`, pero
comparte el kernel del host**. Cuando una imagen declara que su proceso corre
como UID 1000, no está pidiendo "el usuario `node` de mi imagen": está pidiendo
literalmente el entero 1000. Y si en el host ese entero está asignado a una
persona, el kernel no tiene forma de distinguirlos, porque no hay ninguna capa de
traducción entre el contenedor y la máquina.

Dicho de otra forma: el espacio de UID es **uno solo y global**, y lo comparten
tus cuentas de sistema, tus usuarios humanos y todos los contenedores, sin que
nadie coordine el reparto.

## Qué UID eligen realmente las imágenes

Lo interesante es que esto no es aleatorio: las imágenes tiran de un puñado de
números muy predecibles. Puedes ver los que están en uso en cualquier host con
contenedores así:

```bash
ps -eo uid,comm --no-headers | awk '$1>=1000' | sort -n | uniq -c | sort -rn
```

En un clúster de tamaño medio me salieron estos, y cada uno tiene su historia:

| UID | Quién y por qué |
|---|---|
| 1000 | el "primer usuario" de casi cualquier distro. Lo usan las imágenes de Node y muchísimas más |
| 1001 | el segundo usuario. Aparece en aplicaciones que crean su propia cuenta |
| 2000 | número redondo elegido a mano por algunas imágenes, como oauth2-proxy |
| 10000 | otro redondo, típico de herramientas de seguridad |
| 54321 | la imagen oficial de Oracle Database |
| 65532 | el usuario `nonroot` de las imágenes *distroless* de Google |
| 65534 | `nobody`, el UID de desbordamiento que define el propio Linux |
| 65535 | los contenedores `pause` de Kubernetes |

Fíjate en el patrón: **números redondos de cuatro cifras, o el bloque alto de los
65xxx**. Y ese 65532 de distroless no es casualidad; se eligió deliberadamente
alto precisamente para no coincidir con los usuarios estándar de las distros. Es
decir, los que construyen imágenes ya son conscientes del problema y algunos
intentan esquivarlo.

## Por qué esto importa, y por qué no es solo cosmética

Lo primero que rompe es la evidencia forense. Si estás auditando qué ha tocado
una persona, **`find -uid` y `ps -u` dejan de servir como prueba**: te van a
devolver procesos y ficheros de contenedores mezclados con los suyos.

Por suerte hay una salvación, y conviene conocerla: **`auditd` no discrimina por
UID sino por `auid`** (el *loginuid*). PAM lo fija en el momento del login y el
kernel lo hace inmutable, así que sobrevive incluso a un `sudo -i`: un proceso
con `uid=0` y `auid=1001` sigue siendo esa persona escalada a root. Y como los
contenedores no pasan por PAM, tienen el `auid` sin fijar. Por eso esta regla
separa personas de contenedores aunque compartan UID:

```bash
-a always,exit -F arch=b64 -S execve -F auid>=1000 -F auid!=unset -k persona_cmd
```

Las dos mitades son necesarias. `auid>=1000` captura a las personas y
`auid!=unset` excluye a los contenedores; sin la segunda, un host con muchos
contenedores te ahoga el log.

Pero hay un problema que no es cosmético en absoluto: **los permisos efectivos**.
Si un contenedor escribe en un *bind mount* como UID 1001 y tu administrador es
el 1001, esa persona puede leer y modificar esos ficheros, y al revés. No es una
confusión visual, es acceso real. En el caso que me encontré, el binario de un
servicio del sistema figuraba como propiedad de un usuario con `sudo`, lo cual
tiene una lectura desagradable: quien pueda escribir el binario de un servicio
activo controla ese servicio.

> Un ejecutable que arranca como servicio debe ser `root:root`. Si extraes
> releases desde un `.tar.gz` con `tar -xzf` sin más, **preservas el UID del
> empaquetador**, que suele ser 1000 o 1001. Usa `--no-same-owner` o haz un
> `chown` explícito después.
{: .prompt-warning }

## El mapa de rangos de UID en Linux

Si vas a mover tus cuentas a un rango libre, lo primero es saber qué está
reservado. La documentación de systemd tiene la tabla más completa:

| Rango | Para qué |
|---|---|
| 0 | root |
| 1-999 | cuentas de sistema, las asigna la distribución |
| 1000-60000 | usuarios humanos regulares |
| 60001-60513 | usuarios de `systemd-homed` |
| 61184-65519 | usuarios dinámicos (`DynamicUser=` en servicios systemd) |
| 65534 | `nobody` |
| 65535 | `(uid_t) -1` de 16 bits, inutilizable |
| 524288-1879048191 | rangos de UID para contenedores (`systemd-nspawn`) |

Y hay un rango más que no está en esa tabla y que es el que más gente pisa. Míralo
en tu propia máquina:

```bash
grep -E '^SUB_(UID|GID)_(MIN|MAX)' /etc/login.defs
```

Te va a devolver algo así:

```text
SUB_UID_MIN   100000
SUB_UID_MAX   600100000
```

Ese es el rango que `shadow-utils` reparte para los *subordinate IDs*, es decir,
los UID que se delegan a un usuario para que ejecute contenedores sin
privilegios. Cada vez que creas una cuenta con `useradd`, se le asigna
automáticamente un bloque de 65536 UID ahí dentro, y lo puedes ver en
`/etc/subuid`.

> **No metas cuentas de usuario por encima de 100000.** Ese territorio es de los
> subordinate IDs, y solaparlos con UID reales es [CVE-2024-56433]: un
> subordinate ID puede convertirse en el UID efectivo de la cuenta a la que se le
> asignó, vía `newuidmap`. Es un problema conocido de la configuración por
> defecto de `shadow-utils` en entornos con usuarios de red.
{: .prompt-danger }

[CVE-2024-56433]: https://nvd.nist.gov/vuln/detail/CVE-2024-56433

## Entonces, ¿dónde pongo a mis usuarios?

Junta las tres restricciones y la respuesta se estrecha sola:

1. Por encima de donde las imágenes eligen sus UID, que como vimos son redondos
   de cuatro cifras.
2. Por debajo de 61184, donde empiezan los usuarios dinámicos de systemd, y
   desde luego por debajo de 65534.
3. Muy por debajo de 100000, para no acercarse a los subordinate IDs.

Eso deja una banda cómoda entre las **20000 y las 59999**. Yo me quedé con 20000
como base porque es el primer número redondo de cinco cifras que deja un colchón
de 10000 posiciones sobre el territorio de los contenedores, y porque en un
`ls -n` a las tres de la mañana se distingue de un vistazo de cualquier cosa de
cuatro cifras.

Antes de decidirlo, comprueba que la banda está vacía en tus máquinas:

```bash
# cuentas existentes en el rango
awk -F: '$3>=20000 && $3<=59999' /etc/passwd
# y procesos, que es lo que de verdad importa
ps -eo uid= | tr -d ' ' | awk '$1>=20000 && $1<=59999' | sort -u
```

Vale la pena mirar cómo lo resuelven otros, porque la estrategia siempre es la
misma aunque cambien los números. **OpenShift** asigna a cada *namespace* un
rango propio a partir de 1000000000, explícitamente para no pisar usuarios del
host. **DebOps**, un framework de Ansible para gestionar flotas, reserva por
debajo de 1100 para cuentas locales, de 210 a 420 millones para subUID de
contenedores LXC, y a partir de 2000 millones para cuentas de directorio LDAP.
Ninguno inventó nada mágico: **separaron por tipo de identidad en rangos que no
se tocan, y lo escribieron**.

## Migrar cuentas ya existentes: dos trampas

Crear cuentas nuevas en el rango bueno es trivial. Mover las que ya existen tiene
dos sorpresas, y las dos me pillaron.

### `usermod` se niega si hay procesos con ese UID

Esta es la gorda:

```console
# usermod -u 20001 miusuario
usermod: user miusuario is currently used by process 120989
# echo $?
8
```

Y el UID **no cambia**. Lo importante es entender la condición exacta: `usermod`
no comprueba si hay procesos *de esa persona*, comprueba si hay **cualquier
proceso con ese UID**. En una máquina con contenedores usando UID bajos, eso
ocurre casi siempre. Y `userdel` lleva la misma guarda, así que la salida fácil
de "borro y recreo" tampoco funciona.

Puedes anticiparlo antes de intentarlo:

```bash
# ¿hay algo vivo con el UID que quiero liberar?
ps -eo uid= | tr -d ' ' | grep -cx 1001
```

> Si automatizas esto con Ansible, ten en cuenta que **`--check` no lo detecta**.
> En modo simulación el módulo `user` no ejecuta `usermod`, así que te informa
> alegremente de que cambiaría el UID, y el fallo aparece en el apply real.
{: .prompt-warning }

Y hay una consecuencia peor que el propio fallo. La mayoría de roles de gestión
de usuarios crean el **grupo** antes que la cuenta, y `groupmod` **no** tiene esa
guarda. Resultado de un apply a ciegas: el grupo se mueve al GID nuevo, el
usuario se queda en el viejo, y acabas con una cuenta apuntando a un GID que ya no
tiene nombre. Peor que no haber empezado.

### `groupmod` no reetiqueta ficheros, y `usermod -g` tampoco

Esta es más sutil y solo la ves si compruebas el resultado. Después de una
migración aparentemente correcta:

```console
$ ls -ld /home/miusuario
drwx------ 3 miusuario 1001 74 Jul 24 19:04 /home/miusuario
```

El usuario se resuelve bien, pero el **grupo** sale como número. ¿Por qué? Porque
el cambio tiene dos mitades que se comportan distinto:

- `groupmod -g` cambia el número del grupo pero **no toca ni un fichero**.
- `usermod -u` sí reetiqueta el UID de los ficheros del *home*, pero su opción
  `-g` no hace nada aquí, porque compara el **nombre** del grupo primario y el
  nombre no ha cambiado, solo su número.

Así que el *home* queda con el UID nuevo y el GID viejo, ya huérfano. Se arregla
con un `chown` explícito, y conviene hacerlo parte del proceso en lugar de
recordarlo a mano:

```bash
chown -R miusuario:miusuario /home/miusuario
find /home/miusuario -gid 1001 | wc -l    # debe dar 0
```

## Cómo desalojar un UID sin drenar medio clúster

Si el proceso que bloquea el UID está en un pod, no hace falta vaciar el nodo
entero. Basta con localizar el pod concreto y moverlo. El truco es tirar del
cgroup del proceso, que contiene el UID del pod:

```bash
# en el nodo: del PID al UID del pod
for p in $(ps -eo uid,pid --no-headers | awk '$1==1001 {print $2}'); do
  cat /proc/$p/cgroup | grep -oE 'pod[0-9a-f_]{36}' | head -1 | sed 's/^pod//;s/_/-/g'
done
```

Y ese identificador se cruza con la lista de pods:

```bash
kubectl get pods -A \
  -o custom-columns=UID:.metadata.uid,NS:.metadata.namespace,POD:.metadata.name,NODE:.spec.nodeName \
  | grep <el-uid-que-te-salio>
```

Con el pod identificado, el desalojo es un `cordon` del nodo, un `delete` de ese
pod y un `uncordon` al terminar. Mucho menos invasivo que un `drain`, que te
movería también componentes de red o de entrada que quizá no quieras tocar.

> Ojo al orden si tienes varios nodos afectados: los pods que desalojas
> **aterrizan en otro nodo y se llevan el UID conflictivo con ellos**. Deja para
> el final el nodo que los va recibiendo, o acabarás persiguiéndote la cola.
{: .prompt-tip }

## La solución de verdad: user namespaces

Todo lo anterior es gestionar el síntoma. La cura es que el UID del contenedor
deje de ser el UID del host, y eso es exactamente lo que hacen los **user
namespaces**: el kernel mantiene una tabla de traducción por *namespace*, de modo
que un proceso puede creerse el UID 2000 mientras el kernel lo contabiliza como
otro completamente distinto.

En Kubernetes se activa con un campo del *pod spec*, y llegó a GA en la versión
1.36:

```yaml
spec:
  hostUsers: false
```

Lo monté para verlo con mis propios ojos. Un pod pidiendo correr como UID 2000:

```yaml
spec:
  hostUsers: false
  containers:
    - name: probe
      image: busybox:1.36
      command: ["sh","-c","id -u; cat /proc/self/uid_map; sleep 300"]
      securityContext:
        runAsUser: 2000
```

Lo que reporta el contenedor:

```text
2000
         0  739639296      65536
```

Y lo que ve el kernel del nodo para ese mismo proceso:

```console
$ awk '/^Uid:/{print $2}' /proc/<pid>/status
739641296
```

La aritmética cuadra sola: el mapa dice que el UID 0 de dentro es el 739639296 de
fuera, con un rango de 65536. Por tanto 739639296 + 2000 = **739641296**. La
aplicación se cree el 2000, el kernel ve un número altísimo, y **la colisión con
tus usuarios desaparece por construcción sin tocar la imagen**.

Además el kubelet asigna un bloque **distinto a cada pod**, así que también
resuelve algo que normalmente ni nos planteamos: hoy, dos contenedores diferentes
que usen el mismo UID son la misma identidad para el kernel sobre cualquier
volumen compartido. Con *user namespaces*, no.

## Los volúmenes, que es donde estaba el riesgo de verdad

Que el proceso esté remapeado está muy bien, pero **si el proceso se ve como
739641296 y los ficheros del volumen son del 2000, no cuadran**. Aquí es donde
esta funcionalidad ha tardado años en ser usable, y donde había que probar.

Primer intento, con un volumen persistente normal y corriente:

```text
sh: can't create /data/fichero.txt: Permission denied
```

Falla. El pod arranca y el volumen monta, pero no puede escribir. La solución
resultó ser un viejo conocido, `fsGroup`:

```yaml
spec:
  hostUsers: false
  securityContext:
    fsGroup: 2000
    fsGroupChangePolicy: OnRootMismatch
```

Y con eso:

```text
UID=2000 GID=2000 GROUPS=2000
drwxrwsr-x  3 0  2000  4096  /data
ESCRITURA=OK
-rw-r--r--  1 2000 2000  7  f.txt
```

Pero lo verdaderamente elegante está en cómo queda el dato **en disco**. Miré el
mismo fichero desde el sistema de ficheros del nodo:

```console
$ ls -n /var/lib/kubelet/pods/.../f.txt
-rw-r--r-- 1 2000 2000 7 f.txt
```

El fichero es del **2000**, sin remapear, mientras el proceso que lo escribió
corría como un UID de diez cifras. Eso es lo que hacen los **idmap mounts**: la
traducción vive en el montaje, no en el disco. La consecuencia práctica es
enorme: **el volumen sigue siendo portable**. Si mañana quitas `hostUsers: false`,
los datos siguen perteneciendo al UID de siempre y la aplicación los lee igual.
No hay migración de ida ni de vuelta.

## Lo que los user namespaces no cubren

Conviene tener las limitaciones claras antes de ilusionarse:

**No se pueden combinar con los namespaces del host.** El propio apiserver lo
rechaza en admisión, y el mensaje es claro:

```text
The Pod is invalid: spec.hostNetwork: Forbidden: when `hostUsers` is false
```

Lo mismo aplica a `hostPID` y `hostIPC`. Eso deja fuera de entrada a los
componentes de red, a los agentes de almacenamiento y en general a cualquier
DaemonSet privilegiado, que son precisamente los que más te gustaría aislar.

**Los volúmenes en bloque crudo (`volumeDevices`) no funcionan**, y **NFS
tampoco**, porque el cliente NFS de Linux todavía no soporta idmap mounts. Esto
último tiene una consecuencia práctica: si tu almacenamiento implementa los
volúmenes *ReadWriteMany* sobre NFS por debajo, esos volúmenes concretos no van a
poder usar *user namespaces*, aunque los *ReadWriteOnce* del mismo proveedor sí.
Los sistemas de ficheros que sí soportan idmap desde el kernel 6.3 son btrfs,
ext4, xfs, fat, tmpfs y overlayfs.

**Y el kernel se sigue compartiendo.** Un *user namespace* sube mucho el listón,
porque quien escape de un contenedor aterriza en el host como un UID sin
privilegios sobre nada, pero un fallo en un subsistema del kernel se lo salta
igual. No es una máquina virtual.

## Conclusión

Lo que empezó como un `find` con resultados raros terminó siendo una lección
sobre algo que damos por sentado: **la identidad en Linux es un entero global sin
espacios de nombres**, y meter contenedores en esa ecuación sin más significa
compartir ese espacio con procesos que nadie coordina.

Si administras máquinas que corren contenedores, hay dos cosas que puedes hacer
hoy mismo. La primera es comprobar si tienes el problema, que cuesta un comando:

```bash
comm -12 \
  <(awk -F: '$3>=1000 && $3<65000 {print $3}' /etc/passwd | sort -u) \
  <(ps -eo uid= | tr -d ' ' | sort -u)
```

Si eso devuelve algo, tienes UID compartidos entre cuentas del sistema y procesos
en marcha. La segunda es mover tus cuentas humanas a un rango donde nadie más se
siente, documentar por qué elegiste ese rango, y **verificarlo automáticamente**
para que no vuelva a derivar. Ese último paso es el que marca la diferencia entre
arreglarlo una vez y tenerlo arreglado.

Y a medio plazo, los *user namespaces* son el sitio al que va todo esto. Ya no
son experimentales, la traducción funciona, y lo mejor es que no obligan a
reescribir imágenes ni a migrar datos. Solo hace falta paciencia con las
excepciones, que las hay.

## Referencias

- systemd. [*Users, Groups, UIDs and GIDs on systemd systems*](https://github.com/systemd/systemd/blob/main/docs/UIDS-GIDS.md) — la tabla de rangos reservados.
- Kubernetes. [*User Namespaces*](https://kubernetes.io/docs/concepts/workloads/pods/user-namespaces/) — documentación oficial, con la lista de tipos de volumen soportados.
- Kubernetes. [*KEP-127: Support User Namespaces*](https://github.com/kubernetes/enhancements/blob/master/keps/sig-node/127-user-namespaces/README.md) — el diseño y por qué tardó tanto.
- Kubernetes Blog (abril 2025). [*User Namespaces enabled by default*](https://kubernetes.io/blog/2025/04/25/userns-enabled-by-default/).
- Linux kernel. [*Idmappings*](https://docs.kernel.org/filesystems/idmappings.html) — el mecanismo que hace que los volúmenes funcionen.
- shadow-utils. [*login.defs(5)*](https://man7.org/linux/man-pages/man5/login.defs.5.html) — `SUB_UID_MIN` y `SUB_UID_MAX`.
- NVD. [*CVE-2024-56433*](https://nvd.nist.gov/vuln/detail/CVE-2024-56433) — el solape entre subordinate IDs y usuarios de red.
- Red Hat. [*A Guide to OpenShift and UIDs*](https://www.redhat.com/en/blog/a-guide-to-openshift-and-uids).
- DebOps. [*LDAP - POSIX environment integration*](https://docs.debops.org/en/master/ansible/roles/ldap/ldap-posix.html) — su política de rangos UID/GID.
- GoogleContainerTools/distroless. [*Document nonroot and nobody user/group*](https://github.com/GoogleContainerTools/distroless/issues/443) — de dónde sale el 65532.
- The New Stack. [*Kubernetes finally lands user namespace support*](https://thenewstack.io/kubernetes-user-namespace-security/) — el límite del kernel compartido.
