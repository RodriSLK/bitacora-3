# Bitácora Práctica 5: LVM, Respaldo y Recuperación

## 1. El disco nuevo y LVM

Como a `srv1` le estaba faltando espacio, agregamos un disco virtual de 4 GB (`sdb`). En lugar de usar una partición común, elegimos LVM porque nos permite ampliar el almacenamiento más fácilmente si lo necesitamos en el futuro.

Para configurarlo, seguimos estos pasos:

1. Creamos el volumen físico: `sudo pvcreate /dev/sdb`
2. Creamos el grupo de volúmenes `datos`: `sudo vgcreate datos /dev/sdb`
3. Creamos el volumen lógico `respaldos`: `sudo lvcreate -l 100%FREE -n respaldos datos`
4. Le dimos formato `ext4`, creamos la carpeta `/backup` y montamos el volumen.

Después configuramos el montaje automático para que el disco estuviera disponible cada vez que arrancara el servidor. Para eso, buscamos su UUID (`1f7025bf-10d5-4988-9f4e-a793daa3a71d`) y lo agregamos a `/etc/fstab`.

Antes de reiniciar, verificamos la configuración con `sudo findmnt --verify`. Luego reiniciamos y comprobamos con `df -h /backup` que el disco se montara correctamente y tuviera aproximadamente 3.7 GB disponibles.

---

## 2. ¿Qué respaldamos y por qué?

Antes de hacer el respaldo, identificamos los archivos necesarios para que el chequeo automático pudiera seguir funcionando si ocurría algún problema.

Los archivos que guardamos fueron:

- **`/usr/local/bin/healthcheck.sh`**: el script que realiza los chequeos del servidor.
- **`/etc/cron.d/healthcheck`**: la configuración que permite ejecutar el script automáticamente.
- **`/var/log/healthcheck.log`**: el registro donde se guardan los resultados de los chequeos.

Esto nos permitió ver que no alcanza con guardar solamente el registro. También necesitamos conservar el script y la configuración para que el chequeo pueda seguir funcionando.

---

## 3. Creamos el respaldo y guardamos una copia externa

Con el disco nuevo preparado, comprimimos los tres archivos en un único `.tar.gz` y lo guardamos en `/backup`.

Usamos el siguiente comando:

```bash
sudo tar czf /backup/healthcheck-$(date +%F-%H%M).tar.gz /usr/local/bin/healthcheck.sh /etc/cron.d/healthcheck /var/log/healthcheck.log

---

## 4. Simulamos una falla y revisamos qué había pasado

Ejecutamos `intervencion.sh` para simular un problema en el servidor. Cuando terminó, a las 19:29:54, revisamos qué archivos habían sido afectados.

Encontramos lo siguiente:

- El script `healthcheck.sh` seguía presente.
- El registro `healthcheck.log` también seguía existiendo.
- El archivo `/etc/cron.d/healthcheck` había sido eliminado.
- La carpeta local `/backup` también había desaparecido.

Esto nos permitió comprobar que no solamente se había perdido una configuración necesaria para el funcionamiento automático, sino también el respaldo que estaba guardado en el propio servidor.

Como teníamos otra copia en `cli1`. Si no la hubiéramos guardado allí, no habríamos podido recuperar los archivos de esta manera.

---

## 5. Restauramos los archivos y comprobamos que todo funcionara

Para recuperar los archivos, copiamos el respaldo desde `cli1` hacia la carpeta `/tmp` de `srv1`:

```bash
scp sysadmin@192.168.100.100:~/healthcheck-*.tar.gz /tmp/
```

Después, descomprimimos el archivo en la raíz del sistema para que cada archivo volviera a su ubicación original:

```bash
sudo tar xzf /tmp/healthcheck-*.tar.gz -C /
```

Una vez terminada la restauración, usamos `ls -l` para comprobar que `/etc/cron.d/healthcheck` hubiera vuelto a aparecer con sus permisos correspondientes.



### Prueba de funcionamiento

Primero revisamos el registro recuperado. La última entrada correspondía al 26 de septiembre de 2026, a las 20:55:01, fecha en la que se había realizado el respaldo.

Después esperamos unos minutos, sin ejecutar el script manualmente, para ver si cron volvía a realizar el chequeo por su cuenta.

A las 19:35:01 apareció una nueva línea en el registro:

```text
2026-10-13 19:35:01 srv1 OK disco=11% mem=11% ssh=escucha
```

Esto confirmó que la restauración había funcionado y que el chequeo automático volvía a ejecutarse correctamente.

### Resultados

- **Tiempo de recuperación (RTO): 5 minutos y 7 segundos.** Es el tiempo transcurrido desde la falla, a las 19:29:54, hasta que el chequeo volvió a escribir en el registro, a las 19:35:01.
- **Pérdida de datos (RPO):** se perdieron las entradas del registro generadas entre el respaldo del 26 de septiembre y la falla del 13 de octubre. Como esos datos no estaban incluidos en la copia, no pudimos recuperarlos.


---

## 6. Preguntas de cierre

### 1. El respaldo quedó en `cli1`. Las dos máquinas virtuales están en la misma notebook. ¿De qué nos protege esa copia y de qué no?

Tener una copia en `cli1` nos protege ante problemas que afecten solamente a `srv1`, como borrar un archivo por error, dañar una configuración o perder la máquina virtual.

Sin embargo, no nos protege si se rompe el disco físico de la notebook o si el equipo sufre un daño importante, porque las dos máquinas virtuales dependen del mismo hardware.

Por eso, guardar una copia en `cli1` es útil, aunque sería todavía más seguro tener otro respaldo en un equipo o dispositivo diferente.

### 2. El disco nuevo solucionó la falta de espacio. ¿Sirvió para recuperarnos de la falla?

El disco nuevo nos permitió tener más espacio para guardar el respaldo, pero no fue suficiente para recuperarnos de la falla.

Esto ocurrió porque la intervención también eliminó la carpeta `/backup`, donde estaba guardada la copia local. Finalmente, pudimos recuperar los archivos gracias al respaldo que habíamos enviado a `cli1`.

Esto demuestra que guardar una copia en el mismo servidor no siempre es suficiente, aunque esté en un disco diferente.

### 3. ¿Qué pasó con los registros generados entre el respaldo y la falla? ¿Cómo podríamos perder menos información?

Los registros generados después del respaldo y antes de la falla se perdieron porque no estaban incluidos en la copia que restauramos.

Para evitar una pérdida tan grande, podríamos realizar respaldos más seguido, por ejemplo, una vez al día o cada pocas horas, según la importancia de los datos.

Otra opción sería enviar los registros a otro servidor a medida que se generan. Así, si `srv1` tiene un problema, podríamos conservar la información guardada fuera de él.

### 4. ¿Por qué elegimos LVM en lugar de una partición común? ¿Qué podríamos hacer más adelante?

Elegimos LVM porque nos permite administrar y ampliar el espacio de almacenamiento con mayor facilidad.

Por ejemplo, si dentro de unos meses la carpeta `/backup` se queda sin espacio, podríamos agregar otro disco virtual, incorporarlo al grupo `datos` con `vgextend` y ampliar el volumen lógico con `lvextend`.

Esto nos evitaría tener que reorganizar las particiones desde cero. Según la configuración del sistema de archivos, también podríamos ampliar su tamaño sin desmontar el volumen ni apagar el servidor.

### 5. ¿Qué habría pasado si el chequeo hubiera estado escribiendo en el registro mientras ejecutábamos el `tar`?

Si el script hubiera estado escribiendo en el registro mientras hacíamos el respaldo, podría haber ocurrido que la copia no coincidiera exactamente con el estado del archivo en un único momento.

Por ejemplo, el respaldo podría haber incluido solamente una parte de una nueva entrada o haber generado un registro incompleto. Esto no significa que necesariamente se hubiera dañado, pero sí existía ese riesgo.



