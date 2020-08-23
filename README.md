# INSTALACIÓN DE ARCHLINUX CON SOPORTE UEFI #
# DUAL BOOT CON WINDOWS #
Ingrese a https://rootofevilsp.github.io/GIAL-DUAL/ para ver la versión web.

----
*Use el comando:*

    elinks https://rootofevilsp.github.io/GIAL-DUAL/
    
*Para acceder a esta guía desde el Live system de ArchLinux.*

----
Autor: Mario Lourido
Editado por última vez: **21/08/2020 19:00**

El presente documento no pretende ser una guía completa para la instalación de ArchLinux. Es una guía rápida para acelerar el proceso de instalación. Para más detalles, consultar la [**Wiki**](https://wiki.archlinux.org/index.php/Installation_guide_(Espa%C3%B1ol)) de ArchLinux, y su guía de instalación.

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

    *Para listar las distribuciones de teclado disponibles usar:*
    
        ls /usr/share/kbd/keymaps/**/*.map.gz
    
    *Si se desea cargar la distribución para un teclado en español por ejemplo, usar:*
    
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

29. Generar locales:

        nano /etc/locale.gen

    *Descomentar las líneas de interés quitando el símbolo #, en este caso:*

        en_US.UTF-8 UTF-8

    *Guardar presionando Ctrl + X, luego Y y finalmente ENTER*
        
30. Construir el soporte de idioma: 

        locale-gen

31. Crear el archivo de configuración correspondiente:

        nano /etc/locale.conf

    *Agregar el siguiente contenido:*

      LANG=en_US.UTF-8

    *Guardar presionando Ctrl + X, luego Y y finalmente ENTER*

32. Ajustar zona horaria:

        tzselect
        2 
        ENTER
        14 (Número correspondiente a la zona)
        ENTER
        1 (Número correspondiente a la subzona)
        ENTER

33. Borrar el archivo de configuración anterior y crear el link simbólico para hacer el cambio permanente:

        rm /etc/localtime
        ln -s /usr/share/zoneinfo/<ZONA>/<SUB_ZONA> /etc/localtime

    *donde < ZONA > puede ser America y < SUB_ZONA > puede ser Bogota.*
    
34. Instalar **systemd-boot**:

        bootctl --path=/boot install

35. Generar archivo de configuración de systemd-boot:
        
        nano /boot/loader/loader.conf

    Agregar el siguiente contenido:

        default arch
        timeout 3
        editor 0

    *Guardar presionando Ctrl + X, luego Y y finalmente ENTER*

36. Generar el archivo de la entrada por defecto para systemd-boot:

        echo $(blkid -s PARTUUID -o value /dev/sda6) > /boot/loader/entries/arch.conf

    Esto generará un archivo de nombre arch.conf en la ruta especificada, con un contenido similar a:

        14420948-2cea-4de7-b042-40f67c618660

37. Abrir el archivo generado:

        nano /boot/loader/entries/arch.conf

    Se debe agregar lo siguiente, de manera que el serial generado, quede después de PARTUUID y antes de rw, como sigue:

        title ArchLinux
        linux /vmlinuz-linux
        initrd /initramfs-linux.img
        options root=PARTUUID=14420948-2cea-4de7-b042-40f67c618660 rw

    *Guardar presionando Ctrl + X, luego Y y finalmente ENTER*

38. Configuración de red:

    *Agregar el nombre del host a /etc/hostname, por ejemplo:*

        echo gtronick > /etc/hostname

39. Agregar el hostname a /etc/hosts, por ejemplo:

        nano /etc/hosts
        
    *Agregar el siguiente contenido, reemplazando gtronick por tu hostname*
        
        127.0.0.1        localhost.localdomain        localhost
        ::1              localhost.localdomain        localhost
        127.0.1.1        gtronick.localdomain	      gtronick

40. Instalar paquetes para el controlador WiFi:

        pacman -S iw wpa_supplicant dialog elinks vim

41. Ajustar contraseña para  root:

        passwd

    *Ingresar nueva contraseña*   
    *Repetir la contraseña*


42. Salir de la sesión, desmontar particiones:

        exit
        umount -R /mnt
        umount -R /mnt/boot #si existe o aún está montado

43. Antes de reiniciar, verificar que se hayan desmontado todas las particiones de /dev/sda:

        lsblk

44. Por último reiniciar con:

        reboot
  
45. Después de reiniciar el equipo con ArchLinux instalado, crear un nuevo usuario, por ejemplo:

        useradd -m myUser
        
46. Asignar una contraseña al nuevo usuario creado:

        passwd myUser
        
47. Dar permisos de uso para Sudo al nuevo usuario:

        visudo
        
    *Buscar la línea  ROOT  ALL=(ALL) ALL y justo debajo de esta, agregar nuestro usuario, por ejemplo:*
        
        myUser   ALL=(ALL) ALL
    
    *Para editar el documento, presionar la tecla i. Después de esto ya podremos agregar texto normalmente.*
    *Para guardar los cambios, presionar ESC, luego escribir :wq y finalmente ENTER.*
    
48. Probar la conexión de red con:

        ping www.archlinux.org
        
49. Si se presenta error, habilitar e iniciar el servicio de dhcpcd:

        sudo systemctl enable dhcpcd.service
        sudo systemctl start dhcpcd.service
        
     *Se debe tener en cuenta que, si se va a instalar un entorno gráfico, después de instalado se debe deshabilitar el servicio de dhcpcd para poder hacer uso de un administrador de red con interfaz gráfica como NetworkManager*
     
50. Si al tratar de iniciar el sistema, arranca Windows 10 sin mostrar el menu de Systemd-boot, lea la wiki, en el siguiente enlace: 

    https://wiki.archlinux.org/index.php/Unified_Extensible_Firmware_Interface#Windows_changes_boot_order
