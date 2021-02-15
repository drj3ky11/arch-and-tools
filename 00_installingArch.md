Antes de nada, lee la [wiki](https://wiki.archlinux.org/index.php/Installation_guide_(Español))
Y también [aquí](https://denovatoanovato.net/instalar-arch-linux/#establecer-contrasena-del-administrador-root) está maravillosamente explicado todo.

## Inicio de la instalación

    loadkeys es

Para comprobar si BIOS o UEFI:

    ls /sys/firmware/efi/efivars

En esta guía sigo el modo BIOS.
    
## Particionado

    cfdisk
    
**/boot**: aquí va Grub, valen unos 512Mb
**/**: para archivos de programa básicamente, unos 60Gb?
**/swap**: no más de 2G por mucha RAM  que tengas
**/home**: there's no place like home

Para ver las particiones:

    fdisk -l
    
### <u>Formato de las particiones</u>

La /boot en ext2

    mkfs.ext2 /dev/sda1
    
Si la partición que creaste es para UEFI, la formatearás en Fat32.

    mkfs.vfat -F32 /dev/sda1

Raíz y home en ext4

    mkfs.ext4 /dev/sda2
    mkfs.ext4 /dev/sda4

Para la swap, primero le damos formato y luego la activamos:

    mkswap /dev/sda3
    swapon /dev/sda3

### <u>Montando las particiones</u>

    mount /dev/sda2 /mnt
    
Una vez montada la raíz creamos directorios para montar home y boot:

    mkdir /mnt/home
    mkdir /mnt/boot

Montamos las particiones en los directorios creados:

    mount /dev/sda1 /mnt/boot
    mount /dev/sda4 /mnt/home
    
Con UEFI

Primero montaremos la partición raíz

    mount /dev/sda2 /mnt

Ahora creamos los directorios dentro de /mnt para montar las particiones /boot/efi y /home

    mkdir /mnt/home

    mkdir -p /mnt/boot/efi

Y montamos ambas particiones en los directorios creados

    mount /dev/sda1 /mnt/boot/efi

    mount /dev/sda4 /mnt/home
    
## Sistema base
    
Es el momento de comprobar la conexión a internet con un ping a google

Instalarmos el sistema base que lo componen *base* y *base-devel*. También *grub* para el arranque, *networkmanager* para la red. Adicionalmente,para montar usb etc *gvfs* para poder montar un iphone *gvfs-afc* para android *gvfs-mtp*. También intalo *xdg-user-dirs* para crear carpetas por defecto del user de forma automática. También linux, dhcpcd y nano (luego ya vim, pero de momento nano va de sobra)

    pacstrap /mnt base base-devel grub networkmanager gvfs gvfs-afc gvfs-mtp xdg-user-dirs linux linux-firmware nano dhcpcd
    
    
Para la wifi y el touchpad:

    pacstrap /mnt netctl wpa_supplicant dialog
    pacstrap /mnt xf86-input-synaptics
    
    
La tabla de particiones del sistema está en el fichero fstab, hay que crearlo:
    
    genfstab -pU /mnt >> /mnt/etc/fstab
    
## Entrar al sistema base

Para acceder al sistema 

    arch-chroot /mnt
    
Hostname:

    echo nombre > /etc/hostname
    
Zona horaria:

    ln -sf /usr/share/zoneinfo/Europe/Madrid /etc/localtime
    
Para el idioma editamos el /etc/locale.gen, en e caso de Español es es_ES.UTF-8 UTF-8 y generamos el archivo locale.gen:

    locale-gen
    
El reloj hardware se configura:

    hwclock -w
    
Para fijar el teclado:

    echo KEYMAP=es > /etc/vconsole.conf
    
Instalamos Grub (sin UEFI) y creamos el archivo grub.cfg:

    grub-install /dev/sda

    grub-mkconfig -o /boot/grub/grub.cfg
    
Ahora crearemos la pass del root:

    passwd

Y un usuario del montón:

    useradd -m -g users -G audio,lp,optical,storage,video,wheel,games,power,scanner -s /bin/bash nombre_usuario
    
    passwd nombre_usuario

Y ya estaría! Salimos de chroot y desmontamos:

    umount /mnt/boot
    umount /mnt/home
    umount /mnt
    reboot
    
    
Al reiniciar deberíamos editar el /etc/sudoers para dar permisos al usuario que hemos creado y:

    systemctl start NetworkManager.service
    systemctl enable NetworkManager.service
    
Con esto podríamos dejar el usuario root y pasar al normal

## Servidor gráfico

    sudo pacmam -S xorg-server xorg-xinit
    sudo pacman -S mesa mesa-demos
    
Los controladores de video dependerán de la tarjeta:

    lspci | grep VGA
    
**Nvidia**
Nvidia código abierto

    sudo pacman -S xf86-video-nouveau

o

    xf86-video-nv

Desde los Repositorio AUR

Nvidia propietarios
    
    sudo pacman -S nvidia nvidia-utils
    sudo pacman -S nvidia-340xx


**Ultimo recurso en caso de no encontrar**

    sudo pacman -S xf86-video-vesa

Por último el entorno gráfico, en este caso gnome:

    sudo pacman –S gnome gnome–extra

    
El entorno gráfico se puede iniciar mediante un Gestor de inicio también conocidos como Display Manager a elección o iniciarlo manualmente por medio de el comando startx configurándolo según corresponda para cada entorno. Por seguir con el proyecto Gnome:

    sudo pacman -S gdm
    sudo systemctl enable gdm.service
    
    nano /home/"usuario"/.xinitrc
    exec gnome-session

**Y ya estaría!!! Ya tenemos Arch listo.**

##Problema firmas

    sudo pacman -S archlinux-keyring
    sudo pacman -Syu
