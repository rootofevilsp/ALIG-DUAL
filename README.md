----
*Use el comando:*

    elinks https://rootofevilsp.github.io/GIAL-DUAL/
    
*Para acceder a esta guía desde el Live system de ArchLinux.*

----
Autor: Mario Lourido  
Editado por última vez: **28/08/2020 11:10**

El presente documento no pretende ser una guía completa para la instalación de ArchLinux. Es una guía personalizada para acelerar el proceso de instalación sin tener que recurrir a varias fuentes de datos diferentes, y así conseguir con muy poco tiempo y esfuerzo una instalación basica, limpia y funcional.

Pasos preliminares en Windows:

1. Comprobar que Windows está instalado en modo UEFI

    · Arrancar Windows  
    · Pulsar las teclas Win+R para iniciar la ventana ejecutar  
    · En la ventana ejecutar teclear "msinfo32" y pulsar Enter  
    · Se abrirá la ventana "Información del sistema", en ella tendremos que buscar la linea "Modo de BIOS"  
    · Si el valor es UEFI, Windows se ejecutará en modo UEFI-GPT, si el valor es Legacy, se ejecutará en modo BIOS-MBR  

2. Descargar y grabar la ISO de ArchLinux en una memoria USB

    · Descargar la imagen ISO de la pagina oficial de ArchLinux  
    · Descargar "Rufus" para grabar la imagen ISO en una memoria USB arrancable  

3. Libere como mínimo 10 GB de disco duro para la instalación de ArchLinux
    
    · Arranque Windows  
    · Clic derecho sobre "Este equipo"  
    · Clic en Administrar  
    · Clic sobre "Administrador de discos" en el menú de la izquierda  
    · Clic derecho sobre la partición a reducir  
    · Seleccionar "Reducir volumen"  
    · Especificar la cantidad de espacio en MB a reducir  
    · Aceptar y reiniciar  

4. Asegúrese de deshabilitar el fastboot y el secureboot en la configuración de su BIOS

Instalación de ArchLinux:

1. Configurar la BIOS de tu equipo para permitir el arranque desde un dispositivo USB

2. Iniciar la máquina y seleccionar el dispositivo USB de instalación

3. Cuando termine de iniciar estaremos dentro de un terminal "live" lanzado desde el USB, desde aquí instalaremos ArchLinux
    
4. Lo primero que haremos será establecer la distribución de teclado correspondiente. Por defecto la distribución es US

    *Para listar las distribuciones de teclado disponibles:*
    
        ls /usr/share/kbd/keymaps/**/*.map.gz
    
    *Si se desea cargar la distribución para un teclado en español:*
    
        loadkeys es   
        
5. Verificar que estamos en modo UEFI

        ls /sys/firmware/efi/efivars

    *Si se muestra contenido en la carpeta efivars, quiere decir que arrancamos el sistema correctamente en modo UEFI.*
    
6. Verificar la conexión a Internet haciendo ping a una pagina web

        ping google.es

7. Activar la sincronización del reloj del sistema con Internet

        timedatectl set-ntp true

    *Para verificar que se haya sincoronizado correctamente usar:*

        timedatectl status

8. Identificar los discos

        lsblk

9. Verificar la tabla de particiones

        gdisk /dev/sda

    *Se debe listar "GPT Present" al final de la lista.*

*A continuación cada uno debe elegir como particionar su disco, yo he reservado 24 GB en el disco "sda" para instalar ArchLinux y he divido las particiones de las siguiente manera:*

10. Crear particion / *10GB*

        gdisk /dev/sda
        n
        ENTER
        ENTER
        +10G
        8304
        W
        Y

11. Crear partición /home *10GB*

        gdisk /dev/sda
        n
        ENTER
        ENTER
        +10G
        8302
        W
        Y

12. Crear particion swap *4GB*

        gdisk /dev/sda
        n
        ENTER
        ENTER
        +4G
        8200
        W
        Y

    *En cada uno de los tres pasos anteriores te puedes ahorrar las dos ultimas ordenes "W"-"Y", puedes crear todas las particiones de de forma consecutiva y al final ejecutar esas dos ordenes para grabar todo de golpe*

13. Verificar

        lsblk
    
    *Al realizar esta verificación tendremos ya listados los numeros de las particiones "sdaX", a mi me ha quedado como veremos a continuación:*

14. Formatear particion swap

        mkswap /dev/sda7

15. Activar swap

        swapon /dev/sda7

16. Formatear particion / y /home

        mkfs.ext4 /dev/sda5
        mkfs.ext4 /dev/sda6

17. Montar particion / en /mnt
        
        mount /dev/sda5 /mnt
       
18. Crear directorio para /home

        mkdir -p /mnt/home
        
19. Montar partición /home

        mount /dev/sda6 /mnt/home

20. Crear directorio para /boot

        mkdir -p /mnt/boot

21. Montar partición /boot en la partición EFI de Windows

        mount /dev/sda2 /mnt/boot

22. Instalar los paquetes base

        pacstrap /mnt base linux linux-firmware

23. Generar fstab

        genfstab -U /mnt >> /mnt/etc/fstab

24. Verificar

        cat /mnt/etc/fstab

25. Iniciar sesión como root en la instalación

        arch-chroot /mnt /bin/bash

*Llegados a este punto ya estamos dentro de nuestro ArchLinux, pero no reiniciaremos el ordenador porque aun tenemos que configurar ciertas cosas para que al iniciarlo normalmente funcione todo.*

26. Ajustar zona horaria

        ln -sf /usr/share/zoneinfo/<ZONA>/<SUB_ZONA> /etc/localtime

    *Donde <ZONA> puede ser Europa y <SUB_ZONA> puede ser Madrid. Puedes consultarlas con el siguiente comando:*
    
        ls /usr/share/zoneinfo

27. Ajustar el reloj

        hwclock --systohc

28. Generar locales

        nvim /etc/locale.gen

    *Descomentar las líneas de interés quitando el símbolo #, en este caso:*

        es_ES.UTF-8 UTF-8

    *Yo utilizo neovim como editor de texto, pero aquí cada uno usará su editor favorito. Para installar neovim:*
    
        pacman -S neovim
    
29. Construir el soporte de idioma

        locale-gen

30. Agregar el idioma al archivo de configuración correspondiente

        echo "LANG=es_ES.UTF-8" >> /etc/locale.conf

31. Establecer la distribución de teclado en consola de forma permanente

        echo "KEYMAP=es" >> /etc/vconsole.conf

32. Establecer el nombre del host

        echo "rootofevil" >> /etc/hostname

33. Agregar el hostname a /etc/hosts

        nvim /etc/hosts
        
    *Agregar el siguiente contenido, reemplazando rootofevil por tu hostname*
        
        127.0.0.1        localhost
        ::1              localhost
        127.0.1.1        rootofevil.localdomain	      rootofevil

34. Instalar **systemd-boot**

        bootctl --path=/boot install

35. Generar archivo de configuración de systemd-boot
        
        nvim /boot/loader/loader.conf

    *Agregar el siguiente contenido:*

        default arch
        timeout 10
        editor 0

36. Generar el archivo de la entrada por defecto para systemd-boot

        echo $(blkid -s PARTUUID -o value /dev/sda5) > /boot/loader/entries/arch.conf

    *Esto generará un archivo de nombre arch.conf en la ruta especificada, con un contenido similar a:*

        14420948-2cea-4de7-b042-40f67c618660

37. Abrir el archivo generado

        nvim /boot/loader/entries/arch.conf

    *Se debe agregar lo siguiente, de manera que el serial generado, quede después de PARTUUID y antes de rw:*

        title ArchLinux
        linux /vmlinuz-linux
        initrd /initramfs-linux.img
        options root=PARTUUID=14420948-2cea-4de7-b042-40f67c618660 rw

38. Establecer contraseña para root

        passwd

    *Ingresar nueva contraseña*   
    *Repetir la contraseña*

39. Crear un nuevo usuario

        useradd -m rootofevil
        
    *Reemplazando rootofevil por tu nombre de usuario*
        
40. Asignar una contraseña al nuevo usuario creado

        passwd rootofevil

41. Instalar sudo

        pacman -S sudo

42. Dar permisos de uso para Sudo al nuevo usuario

        nvim /etc/sudoers
        
    *Buscar la línea  ROOT  ALL=(ALL) ALL y justo debajo de esta, agregar nuestro usuario:*
        
        rootofevil   ALL=(ALL) ALL
    
    *También se puede editar este documento con el comando:*
    
        visudo
        
    *Usando esta forma se edita sudo con el editor vi, que funciona como vim y neovim, dejo los pasos a seguir por si no se está familiarizado con este editor de texto:*
    *Para editar el documento, presionar la tecla i. Después de esto ya podremos agregar texto normalmente.*
    *Para guardar los cambios, presionar ESC, luego escribir :wq y finalmente ENTER.*

43. Instalar software de red

        pacman -S networkmanager
    
    *Una vez instalado debemos activar el servicio para su ejecución al inicio:*
    
        systemctl enable NetworkManager
        
    *Respetar las mayusculas en este servicio, si no os dará error*

44. Salir de la sesión y desmontar particiones

        exit
        umount -R /mnt
        
    *Al desmontar /mnt debería desmontar el resto automaticamente, en el siguiente paso ya indico que hay que comprobar que todo esta desmontado, si no fuera así repetir este comando editando la ruta a desmontar para que no quede ninguna.

45. Antes de reiniciar, verificar que se hayan desmontado todas las particiones de /dev/sda

        lsblk

46. Por último reiniciar el sistema

        reboot

*Tras reiniciar nos deberia aparecer el menú de systemd para poder seleccionar que sistema operativo queremos arrancar, por defecto arrancará ArchLinux pasados 10". Esto se puede cambiar facilmente pulsando la letra "d" en el porpio menú de systemd, encima del sistema operativo que queremos que se establezca por defecto.*
