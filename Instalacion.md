# Instalación de Arch Linux con UEFI, LVM y GNOME en VirtualBox

## 1. Objetivo

El objetivo de esta práctica fue realizar una instalación manual de **Arch Linux** en una máquina virtual utilizando **VirtualBox**, implementando:

- Disco virtual de **50 GiB**.
- Tabla de particiones **GPT**.
- Arranque mediante **UEFI**.
- Partición `/boot/efi` de **512 MiB**.
- Partición `/boot` de **1 GiB**.
- Administración de almacenamiento mediante **LVM**.
- Volúmenes lógicos separados para `/`, `/usr`, `/var`, `/home` y `swap`.
- No se creó un volumen separado para `/tmp`, siguiendo la recomendación del tutor.
- Gestor de arranque **GRUB**.
- Entorno gráfico **GNOME**.
- Gestor gráfico **GDM**.
- Administración de red mediante **NetworkManager**.
- Herramientas de integración de VirtualBox.

---

# 2. Esquema de almacenamiento

Se utilizó un disco virtual de **50 GiB** identificado como:

```text
/dev/sda
```

La distribución realizada fue:

```text
/dev/sda - 50 GiB
│
├── /dev/sda1 - 512 MiB - FAT32 - /boot/efi
│
├── /dev/sda2 -   1 GiB - EXT4  - /boot
│
└── /dev/sda3 - resto del disco - LVM
     │
     └── vgarch
          │
          ├── root - 15 GiB - /
          ├── usr  - 15 GiB - /usr
          ├── var  -  8 GiB - /var
          ├── home -  6 GiB - /home
          └── swap -  4 GiB - swap
```

El directorio `/tmp` permanece dentro del sistema raíz `/`.

---

# 3. Inicio del medio de instalación

Se configuró VirtualBox para utilizar el ISO de Arch Linux y arrancar en modo UEFI.

En el menú de inicio se seleccionó:

```text
Arch Linux install medium (x86_64, UEFI)
```

Después del arranque se obtuvo la consola:

```text
root@archiso ~#
```

---

# 4. Configuración temporal del teclado

Se configuró el teclado latinoamericano durante la instalación:

```bash
loadkeys la-latin1
```

Este cambio afecta temporalmente al entorno del instalador.

---

# 5. Identificación del disco

Se utilizó:

```bash
lsblk
```

Con este comando se identificó el disco:

```text
/dev/sda
```

de aproximadamente 50 GiB.

---

# 6. Creación de las particiones

Se inició `fdisk`:

```bash
fdisk /dev/sda
```

Se creó una tabla de particiones GPT.

La estructura definida fue:

```text
/dev/sda1   512 MiB   EFI
/dev/sda2     1 GiB   Linux filesystem
/dev/sda3   resto     Linux LVM
```

Se guardaron los cambios mediante:

```text
w
```

Posteriormente se verificó:

```bash
lsblk
```

Obteniendo una estructura similar a:

```text
sda      50G
├─sda1  512M
├─sda2    1G
└─sda3 48.5G
```

---

# 7. Creación del Physical Volume de LVM

La tercera partición fue utilizada para LVM:

```bash
pvcreate /dev/sda3
```

Esto convirtió `/dev/sda3` en un **Physical Volume (PV)**.

La estructura conceptual pasó a ser:

```text
/dev/sda3
    │
    └── Physical Volume
```

---

# 8. Creación del Volume Group

Se creó un Volume Group denominado:

```text
vgarch
```

mediante:

```bash
vgcreate vgarch /dev/sda3
```

La estructura fue:

```text
/dev/sda3
    │
    └── PV
         │
         └── vgarch
```

---

# 9. Creación de los Logical Volumes

Dentro de `vgarch` se crearon los diferentes volúmenes lógicos.

## Swap

```bash
lvcreate -L 4G vgarch -n swap
```

## Raíz

```bash
lvcreate -L 15G vgarch -n root
```

## /usr

```bash
lvcreate -L 15G vgarch -n usr
```

## /var

```bash
lvcreate -L 8G vgarch -n var
```

## /home

```bash
lvcreate -L 6G vgarch -n home
```

Los Logical Volumes quedaron disponibles como:

```text
/dev/vgarch/root
/dev/vgarch/usr
/dev/vgarch/var
/dev/vgarch/home
/dev/vgarch/swap
```

Se verificaron mediante:

```bash
lvs
```

---

# 10. Formateo de las particiones

## Partición EFI

La partición EFI fue formateada como FAT32:

```bash
mkfs.fat -F32 /dev/sda1
```

## Partición /boot

```bash
mkfs.ext4 /dev/sda2
```

## Volumen raíz

```bash
mkfs.ext4 /dev/vgarch/root
```

## Volumen /usr

```bash
mkfs.ext4 /dev/vgarch/usr
```

## Volumen /var

```bash
mkfs.ext4 /dev/vgarch/var
```

## Volumen /home

```bash
mkfs.ext4 /dev/vgarch/home
```

---

# 11. Configuración de swap

El volumen swap se preparó mediante:

```bash
mkswap /dev/vgarch/swap
```

Posteriormente fue activado:

```bash
swapon /dev/vgarch/swap
```

La swap funciona como espacio de intercambio utilizado por Linux como apoyo a la memoria RAM.

---

# 12. Montaje del sistema raíz

Se montó inicialmente el volumen raíz:

```bash
mount /dev/vgarch/root /mnt
```

Durante la instalación, `/mnt` representa el futuro directorio raíz `/` del sistema instalado.

---

# 13. Creación de puntos de montaje

Se crearon los directorios necesarios:

```bash
mkdir -p /mnt/boot
mkdir -p /mnt/usr
mkdir -p /mnt/var
mkdir -p /mnt/home
```

---

# 14. Montaje de /boot

Se montó la segunda partición:

```bash
mount /dev/sda2 /mnt/boot
```

Después se creó el directorio EFI:

```bash
mkdir -p /mnt/boot/efi
```

Es importante crear `/mnt/boot/efi` después de montar `/mnt/boot`.

---

# 15. Montaje de la partición EFI

Se montó:

```bash
mount /dev/sda1 /mnt/boot/efi
```

---

# 16. Montaje de los Logical Volumes

Se montaron los restantes volúmenes:

```bash
mount /dev/vgarch/usr /mnt/usr
```

```bash
mount /dev/vgarch/var /mnt/var
```

```bash
mount /dev/vgarch/home /mnt/home
```

---

# 17. Verificación de los montajes

Se utilizó:

```bash
lsblk
```

La estructura debía presentar aproximadamente:

```text
sda
├─sda1                    /mnt/boot/efi
├─sda2                    /mnt/boot
└─sda3
  ├─vgarch-root           /mnt
  ├─vgarch-usr            /mnt/usr
  ├─vgarch-var            /mnt/var
  ├─vgarch-home           /mnt/home
  └─vgarch-swap           [SWAP]
```

---

# 18. Instalación del sistema base

Se instaló Arch Linux mediante:

```bash
pacstrap -K /mnt base linux linux-firmware lvm2 networkmanager nano sudo grub efibootmgr
```

Los paquetes principales utilizados fueron:

- `base`: componentes básicos de Arch Linux.
- `linux`: kernel Linux.
- `linux-firmware`: firmware para diferentes dispositivos.
- `lvm2`: soporte y administración de LVM.
- `networkmanager`: administración de conexiones de red.
- `nano`: editor de texto.
- `sudo`: ejecución controlada de comandos administrativos.
- `grub`: gestor de arranque.
- `efibootmgr`: administración de entradas UEFI.

---

# 19. Generación de fstab

Se generó el archivo `/etc/fstab` mediante:

```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

Se verificó mediante:

```bash
cat /mnt/etc/fstab
```

El archivo contiene información para montar automáticamente:

```text
/
/boot
/boot/efi
/usr
/var
/home
swap
```

Durante la instalación se detectó una entrada duplicada correspondiente a `/boot`.

Se corrigió editando:

```bash
nano /mnt/etc/fstab
```

y dejando solamente una entrada correspondiente a `/boot`.

---

# 20. Acceso al sistema mediante chroot

Se ingresó al sistema instalado mediante:

```bash
arch-chroot /mnt
```

A partir de este momento los comandos se ejecutaron directamente sobre el nuevo sistema Arch Linux.

---

# 21. Configuración de zona horaria

Para Ecuador se configuró:

```bash
ln -sf /usr/share/zoneinfo/America/Guayaquil /etc/localtime
```

Después se sincronizó el reloj:

```bash
hwclock --systohc
```

---

# 22. Configuración regional

Se editó:

```bash
nano /etc/locale.gen
```

Se habilitaron:

```text
en_US.UTF-8 UTF-8
es_EC.UTF-8 UTF-8
```

Posteriormente se generaron los locales:

```bash
locale-gen
```

Se obtuvo como resultado:

```text
en_US.UTF-8... done
es_EC.UTF-8... done
Generation complete.
```

Se estableció español de Ecuador como configuración principal:

```bash
echo "LANG=es_EC.UTF-8" > /etc/locale.conf
```

---

# 23. Configuración del hostname

Se configuró el nombre del equipo:

```bash
echo "ArchLinuxJob" > /etc/hostname
```

---

# 24. Contraseña del usuario root

Se estableció la contraseña administrativa:

```bash
passwd
```

Se ingresó y confirmó la contraseña solicitada.

---

# 25. Configuración de LVM en mkinitcpio

Esta fue una de las configuraciones más importantes de la instalación.

Primero se verificaron los HOOKS mediante:

```bash
grep '^HOOKS' /etc/mkinitcpio.conf
```

La configuración inicial no incluía correctamente el soporte necesario para LVM.

Se editó:

```bash
nano /etc/mkinitcpio.conf
```

La línea `HOOKS` quedó configurada incluyendo `lvm2`:

```text
HOOKS=(base systemd autodetect microcode modconf kms keyboard sd-vconsole block lvm2 filesystems fsck)
```

La sección crítica fue:

```text
block lvm2 filesystems
```

Esto es necesario porque el sistema raíz está almacenado en:

```text
/dev/vgarch/root
```

Por lo tanto, Linux necesita activar y reconocer LVM durante las primeras etapas del arranque.

---

# 26. Generación del initramfs

Después de configurar LVM se regeneró el initramfs:

```bash
mkinitcpio -P
```

Durante la ejecución apareció:

```text
Running build hook: [lvm2]
```

y finalmente:

```text
Initcpio image generation successful
```

Esto confirmó que el soporte LVM estaba incluido correctamente en el initramfs.

---

# 27. Instalación de GRUB

Se instaló GRUB para sistemas UEFI de 64 bits:

```bash
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=ArchLinux
```

El proceso terminó correctamente con:

```text
Installation finished. No error reported.
```

La opción:

```text
--target=x86_64-efi
```

indica que GRUB debe instalarse para UEFI de 64 bits.

La opción:

```text
--efi-directory=/boot/efi
```

indica dónde está montada la partición EFI.

La opción:

```text
--bootloader-id=ArchLinux
```

establece `ArchLinux` como nombre de la entrada de arranque.

---

# 28. Generación de grub.cfg

Se generó la configuración de GRUB:

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

Este archivo permite que GRUB localice el kernel y el initramfs de Arch Linux.

---

# 29. Verificación del ejecutable EFI

Se comprobó:

```bash
ls /boot/efi/EFI/ArchLinux
```

Se confirmó la existencia de:

```text
grubx64.efi
```

La existencia de este archivo confirmó que GRUB fue instalado correctamente en la partición EFI.

---

# 30. Verificación de la entrada UEFI

Se utilizó:

```bash
efibootmgr
```

Se encontró una entrada similar a:

```text
Boot0003* ArchLinux
```

apuntando a:

```text
\EFI\ArchLinux\grubx64.efi
```

Esto confirmó que el firmware UEFI conocía la ubicación de GRUB.

---

# 31. Instalación de GNOME

Se instaló GNOME junto con GDM:

```bash
pacman -S gnome gdm
```

GNOME proporciona el entorno gráfico del sistema.

GDM proporciona la pantalla gráfica de inicio de sesión.

---

# 32. Habilitación de GDM

Se habilitó GDM:

```bash
systemctl enable gdm
```

Posteriormente se comprobó:

```bash
systemctl is-enabled gdm
```

Resultado:

```text
enabled
```

Esto indica que GDM se ejecutará automáticamente durante el arranque.

---

# 33. Habilitación de NetworkManager

Se ejecutó:

```bash
systemctl enable NetworkManager
```

Esto permite que NetworkManager se inicie automáticamente con Arch Linux.

Para comprobarlo:

```bash
systemctl is-enabled NetworkManager
```

Resultado esperado:

```text
enabled
```

---

# 34. Finalización de la instalación

Se salió del entorno chroot:

```bash
exit
```

Posteriormente se desmontaron recursivamente los sistemas de archivos:

```bash
umount -R /mnt
```

Se desactivó la swap:

```bash
swapoff -a
```

Finalmente se apagó la máquina:

```bash
poweroff
```

---

# 35. Retirada del ISO

Después de apagar la máquina virtual se ingresó a:

```text
VirtualBox
→ Configuración
→ Almacenamiento
```

Se retiró el ISO de Arch Linux de la unidad óptica virtual.

De esta manera la máquina comenzó a arrancar directamente desde el disco virtual.

---

# 36. Secuencia de arranque obtenida

La secuencia final de arranque quedó:

```text
VirtualBox
     │
     ▼
   UEFI
     │
     ▼
/dev/sda1
     │
     ▼
EFI/ArchLinux/grubx64.efi
     │
     ▼
    GRUB
     │
     ▼
Kernel Linux
     │
     ▼
 initramfs
     │
     ▼
    LVM
     │
     ▼
  vgarch
     │
     ▼
vgarch-root
     │
     ▼
Arch Linux
     │
     ▼
    GDM
     │
     ▼
   GNOME
```

---

# 37. Configuración del teclado en GNOME

Después del primer inicio se observó que algunos caracteres especiales del teclado no coincidían correctamente.

Desde GNOME se ingresó a:

```text
Configuración
→ Teclado
→ Fuente de entrada
```

Se seleccionó:

```text
Español (latinoamericano)
```

También se puede configurar desde terminal mediante:

```bash
sudo localectl set-keymap la-latin1
```

```bash
sudo localectl set-x11-keymap latam
```

Para comprobar la configuración:

```bash
localectl status
```

---

# 38. Instalación de VirtualBox Guest Utilities

Después de iniciar correctamente Arch Linux se instalaron las herramientas de integración con VirtualBox:

```bash
sudo pacman -S virtualbox-guest-utils
```

Estas herramientas permiten una mejor integración entre el sistema invitado Arch Linux y VirtualBox.

---

# 39. Habilitación de vboxservice

Se habilitó:

```bash
sudo systemctl enable vboxservice
```

El sistema confirmó la creación del enlace correspondiente a:

```text
vboxservice.service
```

El servicio se ejecutará automáticamente al iniciar Arch Linux.

---

# 40. Actualización del sistema

Una vez terminada la instalación, el sistema puede actualizarse mediante:

```bash
sudo pacman -Syu
```

---

# 41. Comandos de validación

Los siguientes comandos permiten demostrar la configuración realizada.

## Ver particiones y LVM

```bash
lsblk
```

---

## Ver sistemas de archivos

```bash
lsblk -f
```

---

## Ver Physical Volumes

```bash
sudo pvs
```

---

## Ver Volume Groups

```bash
sudo vgs
```

---

## Ver Logical Volumes

```bash
sudo lvs
```

Deben aparecer los volúmenes:

```text
root
usr
var
home
swap
```

dentro de:

```text
vgarch
```

---

## Verificar los puntos de montaje

```bash
findmnt
```

También pueden comprobarse individualmente:

```bash
findmnt /
findmnt /boot
findmnt /boot/efi
findmnt /usr
findmnt /var
findmnt /home
```

---

## Verificar swap

```bash
swapon --show
```

---

## Verificar fstab

```bash
cat /etc/fstab
```

---

## Verificar LVM en HOOKS

```bash
grep '^HOOKS' /etc/mkinitcpio.conf
```

La salida debe contener:

```text
lvm2
```

Por ejemplo:

```text
HOOKS=(base systemd autodetect microcode modconf kms keyboard sd-vconsole block lvm2 filesystems fsck)
```

También se puede validar específicamente mediante:

```bash
grep '^HOOKS' /etc/mkinitcpio.conf | grep lvm2
```

---

## Verificar GRUB

```bash
ls /boot/grub/grub.cfg
```

---

## Verificar el archivo EFI

```bash
ls /boot/efi/EFI/ArchLinux
```

Debe aparecer:

```text
grubx64.efi
```

---

## Verificar entrada UEFI

```bash
efibootmgr
```

Debe aparecer una entrada denominada:

```text
ArchLinux
```

---

## Verificar el escritorio

```bash
echo $XDG_CURRENT_DESKTOP
```

Resultado esperado:

```text
GNOME
```

---

## Verificar GDM

```bash
systemctl is-enabled gdm
```

Resultado esperado:

```text
enabled
```

---

## Verificar NetworkManager

```bash
systemctl is-enabled NetworkManager
```

Resultado esperado:

```text
enabled
```

---

## Verificar VirtualBox Guest Service

```bash
systemctl is-enabled vboxservice
```

Resultado esperado:

```text
enabled
```

---

# 42. Problemas encontrados durante la práctica

## Problema 1: Arch Linux continuaba arrancando desde el ISO

Inicialmente el sistema volvía a presentar:

```text
Arch Linux install medium (x86_64, UEFI)
```

Esto se debía a que el medio ISO continuaba conectado a la máquina virtual.

Antes de retirarlo se verificó que el sistema instalado tuviera correctamente configurados GRUB, UEFI y LVM.

Finalmente se apagó la máquina virtual y se retiró el ISO desde la configuración de VirtualBox.

---

# 43. Problema 2: GRUB EFI no encontrado

Durante uno de los primeros intentos se presentó un problema relacionado con:

```text
\EFI\ArchLinux\grubx64.efi : Not Found
```

Se reinstaló GRUB mediante:

```bash
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=ArchLinux
```

Posteriormente se comprobó:

```bash
ls /boot/efi/EFI/ArchLinux
```

confirmando:

```text
grubx64.efi
```

También se verificó mediante:

```bash
efibootmgr
```

que existiera la entrada UEFI denominada `ArchLinux`.

---

# 44. Problema 3: Kernel Panic con LVM

Durante un primer intento de arranque se produjo el mensaje:

```text
Kernel panic - not syncing: VFS: Unable to mount root fs
```

El kernel comenzaba a arrancar, pero no podía acceder correctamente al sistema raíz.

La raíz se encontraba almacenada dentro de LVM:

```text
/dev/vgarch/root
```

Por este motivo era necesario que el initramfs pudiera reconocer LVM antes de montar `/`.

Se verificó:

```bash
grep '^HOOKS' /etc/mkinitcpio.conf
```

Se agregó:

```text
lvm2
```

a la línea `HOOKS`:

```text
HOOKS=(base systemd autodetect microcode modconf kms keyboard sd-vconsole block lvm2 filesystems fsck)
```

Después se regeneró el initramfs:

```bash
mkinitcpio -P
```

Durante el proceso apareció:

```text
Running build hook: [lvm2]
```

Esto confirmó que el soporte para LVM fue incorporado al initramfs.

Después de esta corrección Arch Linux pudo detectar:

```text
/dev/vgarch/root
```

y montar correctamente el sistema raíz.

---

# 45. Estructura final

El sistema quedó configurado aproximadamente de la siguiente manera:

```text
ARCH LINUX - 50 GiB
│
├── GPT
│
├── UEFI
│
├── /dev/sda1
│     │
│     └── 512 MiB - FAT32
│          │
│          └── /boot/efi
│               │
│               └── EFI/ArchLinux/grubx64.efi
│
├── /dev/sda2
│     │
│     └── 1 GiB - EXT4
│          │
│          └── /boot
│
└── /dev/sda3
      │
      └── LVM Physical Volume
           │
           └── vgarch
                │
                ├── root - 15G → /
                ├── usr  - 15G → /usr
                ├── var  -  8G → /var
                ├── home -  6G → /home
                └── swap -  4G → SWAP
```

---

# 46. Conclusión

Se realizó satisfactoriamente una instalación manual de **Arch Linux** utilizando **GPT, UEFI y LVM** dentro de una máquina virtual de VirtualBox.

El almacenamiento fue organizado separando `/boot/efi` y `/boot` del espacio administrado por LVM.

Dentro del Volume Group `vgarch` se crearon Logical Volumes independientes para:

```text
/
/usr
/var
/home
swap
```

No se creó un volumen independiente para `/tmp`, siguiendo la recomendación establecida para la práctica.

También se configuró correctamente el proceso de arranque mediante:

```text
UEFI
   ↓
GRUB
   ↓
Linux
   ↓
initramfs
   ↓
LVM
   ↓
vgarch-root
   ↓
Arch Linux
   ↓
GDM
   ↓
GNOME
```

Uno de los principales inconvenientes encontrados fue la imposibilidad inicial del kernel para montar la raíz almacenada en LVM.

El problema se solucionó agregando `lvm2` a los HOOKS de:

```text
/etc/mkinitcpio.conf
```

y regenerando el initramfs mediante:

```bash
mkinitcpio -P
```

Finalmente se verificó el funcionamiento de:

- UEFI.
- GRUB.
- LVM.
- Swap.
- GNOME.
- GDM.
- NetworkManager.
- VirtualBox Guest Utilities.

El sistema quedó instalado, configurado y arrancando correctamente desde el disco virtual sin depender del ISO de instalación.
