

# GUIA CACHIPIRULI ASO 3

## Instalación

![Captura desde 2023-03-24 19-43-42](https://user-images.githubusercontent.com/91732612/227621955-5cafd84c-a9ae-4183-aa5b-97a853842ffc.png)
![Captura desde 2023-03-24 20-07-36](https://user-images.githubusercontent.com/91732612/227621969-6577cc75-7e36-4e1c-8a30-80775c8b1007.png)

## SYSLINUX

Instala el paquete de syslinux:
~~~shell
yum install syslinux-efi64
~~~

Crea una carpeta en /boot/efi:
~~~shell
mkdir /boot/efi/EFI/SYSLINUX
~~~

Copia el syslinux a la carpeta nueva:
~~~shell
cp /usr/share/syslinux/efi64/* /boot/efi/EFI/SYSLINUX
~~~

Elimina el syslinux.efi:
~~~shell
rm /boot/efi/EFI/SYSLINUX/syslinux.efi
~~~

Utilizaremos otro syslinux.efi ya que el anterior no funciona, haz el siguiente wget:
~~~shell
su - usuario
wget https://mirrors.edge.kernel.org/pub/linux/utils/boot/syslinux/Testing/6.04/syslinux-6.04-pre1.tar.gz
~~~

Descomprimes el `.tar.gz` y abres la `syslinux-6.04-pre1/efi64/efi/`

~~~shell
cp syslinux.efi /boot/efi/EFI/SYSLINUX
nano /boot/efi/EFI/SYSLINUX/syslinux.cfg
~~~

En ese documento **RECIEN CREADO** ponemos la informacion precisa para la configuracion del syslinux:

**OJO**: en la linea que empieza por APPEND debes poner la partición que coincida con el root, haz un **df -h** para comprobarlo.

~~~shell
DEFAULT fedora
TIMEOUT 20

UI vesamenu.c32

LABEL fedora
 MENU LABEL fedora
 LINUX /EFI/KERNELS/kernelCopy
 INITRD /EFI/KERNELS/rdCopy
 APPEND root=/dev/sda2 ro
~~~

Creamos las carpetas con las copias, de la img y el kernel y ejecutamos

~~~shell
mkdir /boot/efi/EFI/KERNELS
cp /boot/vmlinuz-6.1.18.200.fc37.x86_64 /boot/efi/EFI/KERNELS/kernelCopy
cp /boot/initramfs-6.1.18.200.fc37.x86_64 /boot/efi/EFI/KERNELS/rdCopy
efibootmgr -c -L syslinux -l /EFI/SYSLINUX/syslinux.efi
~~~

> **NOTE**
>
> Si el numero de la versión cambia, no te preocupes y pon el que te aparezca, para ello ejecuta un bonito `ls /boot/`

## Entrada al grub

hacemos `nano /etc/grub.d/40_custom` y ponemos como siempre:

~~~shell
menuentry 'chainloaaderCargador'{
 insmod part_gpt
 insmod chain
	set root=(hd0,gpt1)
	chainloader /EFI/SYSLINUX/syslinux.efi
}
~~~

No existe el `update-grub` en fedora pq UNIX, entonces ponemos eso y el comando:

~~~shell
grub2-mkconfig -o /boot/grub2/grub.cfg
efibootmgr -n 0004
~~~

