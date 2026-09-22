# Ampliación del directorio `/home` en 5 GB utilizando LVM

## Objetivo

Agregar un segundo disco de **5 GB** a Arch Linux y utilizar ese espacio para ampliar el directorio `/home` mediante **LVM (Logical Volume Manager)**.

En este laboratorio el sistema inicialmente tiene:

- Disco principal: `/dev/sda` de 50 GB.
- Partición LVM principal: `/dev/sda3`.
- Volume Group: `vgarch`.
- Logical Volume de `/home`: `vgarch-home`.
- Tamaño inicial de `/home`: 6 GB.
- Disco adicional: `/dev/sdb` de 5 GB.
- Tamaño esperado de `/home` después de la ampliación: aproximadamente 11 GB.

---

## 1. Verificar los discos

Primero se verifica que Arch Linux detecte el nuevo disco de 5 GB.

```bash
lsblk
```

Se debe observar una estructura similar:

```text
sda                         50G
├─sda1                     512M  /boot/efi
├─sda2                       1G  /boot
└─sda3                    48.5G
  ├─vgarch-swap               4G  [SWAP]
  ├─vgarch-root              15G  /
  ├─vgarch-usr               15G  /usr
  ├─vgarch-var                8G  /var
  └─vgarch-home               6G  /home

sdb                          5G
```

El disco:

```text
/dev/sdb
```

corresponde al nuevo disco de **5 GB**.

> **Importante:** No se modificará `/dev/sda`, `/dev/sda1`, `/dev/sda2` ni `/dev/sda3`, ya que el espacio adicional proviene del nuevo disco `/dev/sdb`.

---

## 2. Verificar la configuración actual de LVM

Ejecutar:

```bash
pvs
```

Luego:

```bash
vgs
```

Y:

```bash
lvs
```

Estos comandos permiten verificar:

- **PVS:** Physical Volumes.
- **VGS:** Volume Groups.
- **LVS:** Logical Volumes.

En este laboratorio el Volume Group se llama:

```text
vgarch
```

y el Logical Volume correspondiente a `/home` es:

```text
/dev/vgarch/home
```

---

## 3. Convertir el nuevo disco en Physical Volume

Para que LVM pueda utilizar `/dev/sdb`, primero se convierte el disco en un **Physical Volume (PV)**.

Ejecutar:

```bash
pvcreate /dev/sdb
```

Después verificar:

```bash
pvs
```

Ahora `/dev/sdb` debe aparecer como un Physical Volume.

---

## 4. Agregar el nuevo disco al Volume Group

El siguiente paso consiste en incorporar `/dev/sdb` al Volume Group existente llamado `vgarch`.

Ejecutar:

```bash
vgextend vgarch /dev/sdb
```

Verificar:

```bash
pvs
```

Y:

```bash
vgs
```

Ahora el Volume Group `vgarch` dispone del espacio proveniente de:

```text
/dev/sda3
```

y:

```text
/dev/sdb
```

Conceptualmente:

```text
/dev/sda3 ──────┐
                │
                ├──── vgarch
                │
/dev/sdb  ──────┘
   5 GB
```

---

## 5. Extender el Logical Volume de `/home`

El directorio `/home` utiliza:

```text
/dev/vgarch/home
```

Inicialmente tiene aproximadamente:

```text
6 GB
```

Se agregan **5 GB** ejecutando:

```bash
lvextend -L +5G /dev/vgarch/home
```

La opción:

```text
-L +5G
```

significa **agregar 5 GB al tamaño existente**.

Por lo tanto:

```text
6 GB + 5 GB = 11 GB
```

Verificar el nuevo tamaño:

```bash
lvs
```

El volumen `home` debe aparecer ahora con aproximadamente:

```text
11.00g
```

---

## 6. Ampliar el sistema de archivos

Aumentar el Logical Volume no significa automáticamente que el sistema de archivos esté utilizando todo el nuevo espacio.

Primero se verifica el sistema de archivos de `/home`:

```bash
df -Th /home
```

Si `/home` utiliza **ext4**, se amplía mediante:

```bash
resize2fs /dev/vgarch/home
```

Este comando hace que el sistema de archivos ext4 utilice el nuevo espacio disponible en el Logical Volume.

---

## 7. Verificación final

Verificar los Physical Volumes:

```bash
pvs
```

Verificar el Volume Group:

```bash
vgs
```

Verificar los Logical Volumes:

```bash
lvs
```

Finalmente verificar el tamaño disponible en `/home`:

```bash
df -h /home
```

El volumen `/home` debe mostrar aproximadamente **11 GB** de capacidad total.

---

# Comandos utilizados

La secuencia principal utilizada en la práctica fue:

```bash
lsblk
pvs
vgs
lvs
```

Crear el Physical Volume:

```bash
pvcreate /dev/sdb
```

Agregar el nuevo disco al Volume Group:

```bash
vgextend vgarch /dev/sdb
```

Verificar:

```bash
pvs
vgs
```

Ampliar `/home`:

```bash
lvextend -L +5G /dev/vgarch/home
```

Ampliar el sistema de archivos ext4:

```bash
resize2fs /dev/vgarch/home
```

Verificación final:

```bash
lvs
df -h /home
```

---

# ¿Qué es LVM?

**LVM** significa **Logical Volume Manager**.

LVM permite administrar el almacenamiento de Linux de forma flexible y separa el almacenamiento físico de los volúmenes utilizados por el sistema operativo.

LVM trabaja principalmente con tres conceptos:

## PV - Physical Volume

Un Physical Volume es un disco o partición preparado para ser utilizado por LVM.

En este laboratorio:

```text
/dev/sda3
/dev/sdb
```

son utilizados como Physical Volumes.

El nuevo `/dev/sdb` se convirtió en PV mediante:

```bash
pvcreate /dev/sdb
```

---

## VG - Volume Group

Un **Volume Group** agrupa uno o varios Physical Volumes en un conjunto común de almacenamiento.

En este laboratorio el VG se llama:

```text
vgarch
```

Después de agregar el segundo disco, conceptualmente queda:

```text
Physical Volumes

/dev/sda3 ─────────┐
                   │
                   ├── VG: vgarch
                   │
/dev/sdb 5 GB ─────┘
```

El comando utilizado fue:

```bash
vgextend vgarch /dev/sdb
```

---

## LV - Logical Volume

Los **Logical Volumes** son divisiones creadas dentro del Volume Group.

En este sistema existen volúmenes como:

```text
vgarch-root
vgarch-usr
vgarch-var
vgarch-home
vgarch-swap
```

Cada volumen puede utilizarse para un punto de montaje diferente.

Por ejemplo:

```text
vgarch-root  → /
vgarch-usr   → /usr
vgarch-var   → /var
vgarch-home  → /home
vgarch-swap  → swap
```

---

# ¿Qué se hizo en esta práctica?

Se agregó un segundo disco de **5 GB** al sistema.

El nuevo disco fue detectado como:

```text
/dev/sdb
```

El procedimiento realizado fue:

1. Se verificó el nuevo disco mediante `lsblk`.
2. Se convirtió `/dev/sdb` en un Physical Volume mediante `pvcreate`.
3. Se agregó `/dev/sdb` al Volume Group `vgarch` mediante `vgextend`.
4. Se agregaron 5 GB al Logical Volume correspondiente a `/home` mediante `lvextend`.
5. Se amplió el sistema de archivos ext4 mediante `resize2fs`.
6. Se verificó el nuevo tamaño mediante `lvs` y `df -h`.

El resultado final fue aproximadamente:

```text
/home

Antes:    6 GB
Agregado: 5 GB
----------------
Después: 11 GB
```

---

# Esquema del procedimiento

```text
SEGUNDO DISCO
/dev/sdb
5 GB
   │
   │ pvcreate
   ▼
Physical Volume
   │
   │ vgextend
   ▼
VG: vgarch
   │
   │ lvextend +5G
   ▼
LV: vgarch-home
   │
   │ resize2fs
   ▼
/home
6 GB → 11 GB
```

---

# Conclusión

Mediante **LVM** fue posible agregar un segundo disco de 5 GB y utilizar su capacidad para ampliar el volumen lógico correspondiente al directorio `/home`.

La principal ventaja observada es que el almacenamiento pudo ampliarse sin reinstalar Arch Linux y sin modificar las particiones de arranque `/boot` y `/boot/efi`.

El nuevo disco `/dev/sdb` fue agregado al Volume Group `vgarch`, y posteriormente el espacio adicional fue asignado al Logical Volume de `/home`, aumentando su capacidad de aproximadamente **6 GB a 11 GB**.