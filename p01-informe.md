# Informe inicial del servidor — <grupo 3 >

**Integrantes:** <Sanz, Rodrigo>

> Plantilla del informe de P01. Las secciones 1 y 2 se completan EN CLASE.
> Las secciones 3 y 4 pueden completarse después. Cada dato lleva el comando
> que lo produjo — un dato sin comando no es evidencia.
>
> **Cada bloque de evidencia lleva quién lo ejecutó.** El informe es del grupo,
> pero el servidor es de cada uno: la columna "Ejecutó" dice sobre qué ecosistema
> se corrió el comando. La salida tiene que mostrar el **host** y una **marca de
> tiempo** coherentes con ese alumno y esa fecha (los trae `hostnamectl`, el
> prompt o un `date` al lado). **Evidencia sin autor no puntúa** — ver la
> **rúbrica del curso**, sección «Trabajo individual y grupal».

## 1. Inventario (evidencia)

| Pregunta del gerente              | Respuesta                                            | Comando    | Ejecutó                    |
|---------------------------------- |------------------------------------------------------|------------|--------------------------- |
| ¿Qué máquina/SO es?               |base/ Debian12 (bookworn) Linux 6.1.0-52              |hostnamectl |cli1 conectado a srv1- Sanz |
| ¿Qué direcciones tiene?           |enp0s3: 10.0.2.15/24, enp0s8:192.168.100.10/24        |ip -br a    |cli1 conectado a srv1- Sanz |
| ¿Cuánto disco hay y cuánto queda? |Total sda20G./dev/sda1 19G(Usado: 1.9G, Libre 16G,11%)|lsblk y df-h|cli1 conectado a srv1- Sanz |
| ¿Cuánta memoria?                  |Mem Ttoal: 1.9Gi, Usado:236Mi, Libre:1.7Gi            |free -h     |cli1 conectado a srv1- Sanz |
| ¿Qué servicios corren? (lista)    |cron,dbus,getty@tty1,ssh,systemd-journald,            |systemctl   |cli1 conectado a srv1- Sanz |
| Continua la lista                 |systemd-logind.systemd-timesyncd,systemd-udevd,       |            |                            |
|                                   |user@1000,wpa_suplicant                               |            |                            |
| ¿Quién entró últimamente?         |sysadmin desde 192.168.100.100 en pts/0 y tty1        |last        |cli1 conectado a  srv1-Sanz |

> La columna **Comando** la completás vos: es la mitad de la evidencia. Una
> respuesta sin el comando que la produjo no puntúa.

## 2. Navegación (evidencia)

- ¿Existe `/etc/ssh/sshd_config`? Y de las entradas directamente dentro de
  `/etc`, ¿cuál cambió más recientemente?
  - Respuesta / comando: Si existe ( ls -l /etc/ssh/sshd_config). La mas reciente es resolv-conf ( ls -lt /etc | head)
  - Ejecutó (nombre y VM): Sanz Rodrigo - cli1 conectado a srv1
- Últimos diez eventos del servicio SSH — ¿qué registran?
  - Respuesta / comando: sudo journalctl -u ssh -n 10 
  - Explicacion: Registra la detencion o inicio del servicio sshd y las conexiones SSH aceptadas mediante contrasela para el usuario sysadmin
  - Ejecutó (nombre y VM): Sanz Rodrigo - cli1 conectado a srv1

## 3. Problemas encontrados

(qué falló, qué escalón de la escalera dio la evidencia, cómo se resolvió)

### La falla inyectada

| | |
|---|---|
| **Síntoma** — qué se veía | |
| **Escalón** donde apareció la evidencia decisiva | |
| **Salida de ese escalón** (pegada) | |
| **Causa** — qué estaba roto de verdad | |
| **Arreglo** — qué se hizo | |
| **Quién la encontró** | |

> La última fila no es un trámite: el criterio pide que el grupo pueda decir
> **quién** bajó la escalera hasta la evidencia. Y lo que se corrige no es que el
> servidor vuelva a andar, sino que puedan explicar **por qué dejó de andar y
> cómo lo supieron**.

## 4. Análisis y conclusión

(respuestas a las preguntas de cierre + conclusión del estado del servidor)

## 5. Quién hizo qué

Se completa en todos los informes; sirve para verificar la rotación al cierre del
semestre (cada integrante tiene que haber firmado al menos 3 prácticas).

| Integrante | De qué se ocupó en esta práctica |
|------------|----------------------------------|
| | |
| | |
| | |
