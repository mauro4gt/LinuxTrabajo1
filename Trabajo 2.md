# Trabajo 2: Script de administración y monitoreo en Arch Linux

## 1. Objetivo

Desarrollar un script en Bash para realizar un monitoreo básico del sistema operativo Arch Linux.

El script permite verificar:

- Uso del almacenamiento de un directorio.
- Memoria disponible.
- Estado del servicio SSH.
- Tiempo de actividad del sistema.
- Nombre del equipo.
- Estado general del sistema.
- Validación de directorios ingresados como parámetro.
- Generación automática de un archivo de reporte.

---

## 2. Entorno utilizado

Para realizar la práctica se utilizó una máquina virtual con Arch Linux.

El script fue trabajado desde el usuario:

```bash
netadmin
```

La ubicación utilizada para almacenar el script fue:

```bash
/home/netadmin/scripts
```

---

## 3. Verificación del usuario

Primero se verificó el usuario conectado al sistema mediante:

```bash
whoami
```

El resultado esperado fue:

```text
netadmin
```

---

## 4. Creación del directorio para almacenar el script

Se creó el directorio `scripts` dentro del directorio personal del usuario `netadmin`.

```bash
mkdir -p /home/netadmin/scripts
```

Luego se ingresó al directorio:

```bash
cd /home/netadmin/scripts
```

Para comprobar la ubicación actual se utilizó:

```bash
pwd
```

Resultado:

```text
/home/netadmin/scripts
```

---

## 5. Creación del script

Dentro del directorio `scripts` se creó el archivo:

```text
monitor_sistema.sh
```

Se utilizó el editor Nano:

```bash
nano monitor_sistema.sh
```

Dentro del archivo se agregó el siguiente script:

```bash
#!/bin/bash

# ==========================================================
# Deber 2 - Script de Administracion en Bash
# Monitoreo basico de un sistema Arch Linux
# ==========================================================

# Directorio recibido como parametro.
# Si no se proporciona, se utiliza /.
directorio="${1:-/}"

# Archivo de reporte
reporte="reporte_sistema.txt"

# Estado general
estado_general=0

# Validar que el directorio exista
if [ ! -d "$directorio" ]; then
    echo "ERROR: El directorio '$directorio' no existe."
    exit 1
fi

# Informacion general del sistema
fecha=$(date '+%F %T')
equipo=$(cat /etc/hostname)
tiempo_actividad=$(uptime -p)

# Verificacion del almacenamiento
uso_disco=$(df -P "$directorio" | awk 'NR==2 {gsub("%","",$5); print $5}')

if [ "$uso_disco" -ge 90 ]; then
    estado_disco="ERROR"
    estado_general=1
else
    estado_disco="OK"
fi

# Verificacion de memoria
memoria_disponible=$(free -m | awk '/^Mem:/ {print $7}')

if [ "$memoria_disponible" -lt 500 ]; then
    estado_memoria="ERROR"
    estado_general=1
else
    estado_memoria="OK"
fi

# Verificacion del servicio SSH
servicio="sshd"

if systemctl is-active --quiet "$servicio"; then
    estado_servicio="ACTIVO - OK"
else
    estado_servicio="NO ACTIVO - ERROR"
    estado_general=1
fi

# Determinar resultado general
if [ "$estado_general" -eq 0 ]; then
    resultado_general="SIN ERRORES"
else
    resultado_general="SE DETECTARON ERRORES"
fi

# Generar reporte
echo "REPORTE DE MONITOREO DEL SISTEMA" > "$reporte"
echo "Fecha: $fecha" >> "$reporte"
echo "Equipo: $equipo" >> "$reporte"
echo "Directorio analizado: $directorio" >> "$reporte"
echo "Uptime: $tiempo_actividad" >> "$reporte"
echo "Uso de almacenamiento ($directorio): ${uso_disco}% - $estado_disco" >> "$reporte"
echo "Memoria disponible: ${memoria_disponible} MiB - $estado_memoria" >> "$reporte"
echo "Servicio $servicio: $estado_servicio" >> "$reporte"
echo "RESULTADO GENERAL: $resultado_general" >> "$reporte"

# Mostrar reporte en pantalla
echo
cat "$reporte"
```

---

## 6. Funcionamiento del parámetro del script

El script permite recibir un directorio mediante el primer parámetro `$1`.

La siguiente instrucción realiza esta función:

```bash
directorio="${1:-/}"
```

Esto significa que si se ejecuta:

```bash
./monitor_sistema.sh /home
```

el valor de `$1` será:

```text
/home
```

Por lo tanto, el script analizará `/home`.

Si no se proporciona ningún parámetro:

```bash
./monitor_sistema.sh
```

se utilizará automáticamente:

```text
/
```

como directorio predeterminado.

---

## 7. Validación del directorio

Antes de realizar el monitoreo, el script comprueba que el directorio ingresado realmente exista.

La validación utilizada fue:

```bash
if [ ! -d "$directorio" ]; then
    echo "ERROR: El directorio '$directorio' no existe."
    exit 1
fi
```

Si la ruta no existe, el script muestra un mensaje de error y finaliza de manera controlada.

---

## 8. Verificación del almacenamiento

Para obtener información sobre el almacenamiento se utilizó:

```bash
df -P "$directorio"
```

Posteriormente se utilizó `awk` para obtener únicamente el porcentaje de utilización:

```bash
uso_disco=$(df -P "$directorio" | awk 'NR==2 {gsub("%","",$5); print $5}')
```

La validación establece:

- Uso menor al 90%: `OK`
- Uso igual o mayor al 90%: `ERROR`

La condición utilizada fue:

```bash
if [ "$uso_disco" -ge 90 ]; then
    estado_disco="ERROR"
    estado_general=1
else
    estado_disco="OK"
fi
```

---

## 9. Verificación de memoria RAM

Para conocer la memoria disponible se utilizó:

```bash
free -m
```

El valor de memoria disponible se obtuvo mediante:

```bash
memoria_disponible=$(free -m | awk '/^Mem:/ {print $7}')
```

El script utiliza como límite 500 MiB.

La validación funciona de la siguiente forma:

- 500 MiB o más disponibles: `OK`
- Menos de 500 MiB disponibles: `ERROR`

La condición implementada fue:

```bash
if [ "$memoria_disponible" -lt 500 ]; then
    estado_memoria="ERROR"
    estado_general=1
else
    estado_memoria="OK"
fi
```

Durante la prueba realizada, la máquina virtual presentó aproximadamente:

```text
207 MiB
```

de memoria disponible.

Por este motivo el resultado fue:

```text
Memoria disponible: 207 MiB - ERROR
```

Este resultado confirma que la condición implementada en el script funciona correctamente.

---

## 10. Verificación del servicio SSH

El servicio utilizado para la práctica fue:

```text
sshd
```

Inicialmente se verificó mediante:

```bash
systemctl status sshd
```

El servicio se encontraba:

```text
inactive (dead)
```

Además, estaba configurado como:

```text
disabled
```

Para habilitar e iniciar el servicio se ejecutó:

```bash
sudo systemctl enable --now sshd
```

Posteriormente se comprobó mediante:

```bash
systemctl is-active sshd
```

Resultado:

```text
active
```

Dentro del script se realiza automáticamente la comprobación con:

```bash
if systemctl is-active --quiet "$servicio"; then
    estado_servicio="ACTIVO - OK"
else
    estado_servicio="NO ACTIVO - ERROR"
    estado_general=1
fi
```

---

## 11. Obtención del nombre del equipo

Inicialmente se utilizó:

```bash
hostname
```

Sin embargo, el sistema presentó:

```text
hostname: orden no encontrada
```

Para solucionar el inconveniente se obtuvo directamente el nombre configurado en `/etc/hostname`:

```bash
equipo=$(cat /etc/hostname)
```

El resultado obtenido en la máquina fue:

```text
ArchLinuxJob
```

---

## 12. Obtención del tiempo de actividad

Para mostrar cuánto tiempo lleva activo el sistema se utilizó:

```bash
uptime -p
```

Este valor es únicamente informativo y no modifica el resultado general del monitoreo.

Durante una de las pruebas se obtuvo:

```text
Uptime: up 30 minutes
```

---

## 13. Asignación de permisos de ejecución

Una vez creado el archivo, se proporcionó permiso de ejecución:

```bash
chmod +x monitor_sistema.sh
```

Los permisos se comprobaron mediante:

```bash
ls -l monitor_sistema.sh
```

El archivo presentó permisos similares a:

```text
-rwxr-xr-x
```

Las letras `x` indican que el archivo dispone de permiso de ejecución.

---

## 14. Comprobación de sintaxis

Antes de ejecutar el script se verificó su sintaxis mediante:

```bash
bash -n monitor_sistema.sh
```

Al no obtener ningún mensaje de error, se confirmó que la sintaxis del script era correcta.

---

# 15. Pruebas realizadas

## 15.1 Prueba con el directorio /home

Se ejecutó:

```bash
./monitor_sistema.sh /home
```

Durante la prueba se obtuvo un resultado similar al siguiente:

```text
REPORTE DE MONITOREO DEL SISTEMA
Fecha: 2026-09-22 09:09:28
Equipo: ArchLinuxJob
Directorio analizado: /home
Uptime: up 30 minutes
Uso de almacenamiento (/home): 1% - OK
Memoria disponible: 207 MiB - ERROR
Servicio sshd: ACTIVO - OK
RESULTADO GENERAL: SE DETECTARON ERRORES
```

El almacenamiento presentó únicamente un 1% de utilización, por lo que obtuvo estado `OK`.

El servicio `sshd` también se encontraba activo.

La memoria disponible era inferior a 500 MiB, por lo que se obtuvo estado `ERROR`.

Como existe al menos una condición de error, el resultado general fue:

```text
SE DETECTARON ERRORES
```

### Evidencia

Agregar aquí la captura correspondiente a la ejecución:

```markdown
images/script-home.png
```

---

## 15.2 Prueba con el directorio /var

Se realizó una segunda ejecución utilizando:

```bash
./monitor_sistema.sh /var
```

En la prueba realizada se observó un uso aproximado de almacenamiento del:

```text
22%
```

Por lo tanto, el almacenamiento obtuvo:

```text
OK
```

La memoria disponible continuó por debajo del límite establecido, por lo que el resultado general indicó la existencia de errores.

### Evidencia

```markdown
images/script-var.png
```

---

## 15.3 Prueba de directorio inexistente

Para verificar el control de errores se utilizó una ruta inexistente:

```bash
./monitor_sistema.sh /noexiste
```

El resultado esperado es:

```text
ERROR: El directorio '/noexiste' no existe.
```

Esta prueba demuestra que el script valida la existencia del directorio antes de continuar con el monitoreo.

### Evidencia

```markdown
images/script-error-directorio.png
```

---

## 15.4 Prueba sin parámetros

También se ejecutó el script sin proporcionar ningún directorio:

```bash
./monitor_sistema.sh
```

En este caso el script utiliza automáticamente:

```text
/
```

Esto ocurre gracias a:

```bash
directorio="${1:-/}"
```

### Evidencia

```markdown
images/script-sin-parametro.png
```

---

# 16. Generación del archivo de reporte

El script genera automáticamente:

```text
reporte_sistema.txt
```

Para comprobar su existencia se utilizó:

```bash
ls -l reporte_sistema.txt
```

El contenido se puede visualizar mediante:

```bash
cat reporte_sistema.txt
```

El script utiliza el operador `>` para crear o sobrescribir inicialmente el archivo:

```bash
echo "REPORTE DE MONITOREO DEL SISTEMA" > "$reporte"
```

Posteriormente utiliza `>>` para agregar información:

```bash
echo "Fecha: $fecha" >> "$reporte"
```

De esta forma se genera automáticamente el reporte cada vez que se ejecuta correctamente el script.

### Evidencia

```markdown
images/reporte-sistema.png
```

---

# 17. Comandos principales utilizados

Los principales comandos utilizados durante el desarrollo fueron:

```bash
mkdir
cd
pwd
nano
chmod
ls
cat
date
uptime
df
free
awk
systemctl
bash
```

También se utilizaron estructuras propias de Bash como:

```bash
if
else
fi
```

y variables como:

```bash
$1
$directorio
$uso_disco
$memoria_disponible
$estado_general
```

---

# 18. Resultado final

El script desarrollado permite realizar un monitoreo básico de Arch Linux utilizando Bash.

Se consiguió implementar:

- Recepción de directorios mediante parámetros.
- Directorio `/` como valor predeterminado.
- Validación de la existencia del directorio.
- Monitoreo del almacenamiento.
- Monitoreo de memoria disponible.
- Verificación del servicio `sshd`.
- Obtención del tiempo de actividad.
- Obtención del nombre del equipo.
- Clasificación mediante estados `OK` y `ERROR`.
- Resultado general del monitoreo.
- Generación automática del archivo `reporte_sistema.txt`.
- Visualización del reporte directamente en la terminal.

Durante las pruebas se comprobó que el sistema de almacenamiento y el servicio `sshd` presentaban un estado correcto. Sin embargo, la cantidad de memoria disponible era inferior al límite de 500 MiB establecido para la práctica, por lo cual el script identificó correctamente esta condición como un error.

---

# 19. Conclusión

La práctica permitió aplicar conceptos fundamentales de administración de sistemas GNU/Linux mediante scripts Bash.

Se utilizaron variables, parámetros, sustitución de comandos, estructuras condicionales, comparaciones numéricas, validación de directorios, redirección de salida y comandos propios de administración del sistema.

Las pruebas realizadas permitieron comprobar que el script responde correctamente tanto ante condiciones normales como ante errores. Además, la generación del archivo `reporte_sistema.txt` permite conservar el resultado del último monitoreo realizado.

El desarrollo demuestra cómo Bash puede utilizarse para automatizar tareas repetitivas de supervisión y administración en un sistema Arch Linux.