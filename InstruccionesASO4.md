# Guía Chachipiruli ASO 4

> **WARNING**
>
> Antes de tocar nada de esta práctica recomendamos asertivamente que clones la maquina en la que vas a configurar las movidas (por si acaso), cuando clones acuerdate de que se mantengan las UUID o los nombres de disco, no queremos tener que joder la P2

## Ubuntu

~~~shell
apt update
apt install ubuntu-mate-core
~~~

## DEVUAN 

Para instalar MATE y LightDM ejecutamos:
~~~~~shell
sudo apt install mate-desktop-environment
sudo apt install lightdm lightdm-gtk-greeter
~~~~~

Para poner MATE como entorno gráfico por defecto en nuestra máquina, ejecutamos ( si tenemos que cambiar el entorno gráfico por cualquier tipo de problema, volveríamos a ejecutar el mismo comando):
~~~~~shell
sudo update-alternatives --config x-session-manager
# y seleccionamos el nº correspondiente a MATE
~~~~~

Después de hacer reboot, para comprobar que todo está OK se puede hacer:
~~~~shell
echo $XDG_CURRENT_DESKTOP #nos tiene que devolver MATE
cat /etc/X11/default-display-manager #tiene que devolver /usr/sbin/lightdm
~~~~

Para las VBoxGuestAdditions ->
~~~~shell
sudo apt install build-essential dkms linux-headers-$(uname -r)
	añades cd en devices
	montas el cd ->OPCIONAL PERO NO NECESARIO (puedes añadirlo desde entorno grafico)
  entras a la carpeta y ejecutas
  sh ./VBoxLinuxGuestAdditions.run 
  
~~~~

## FEDORA

Ya viene instalado que yo sepa

## SOLARIS
#### Si no te funciona el ping
configurar el adaptador intel (igual que con la p2)
~~~shell
dladm show-phys
~~~
Con la red que te salga (en mi caso net0)
~~~shell
ipadm create-ip net0
ipadm create-addr -T dhcp net0
reboot
~~~
Denada
### INSTALAR GUEST ADDITIONS:

Al insertar la iso (devices->insert guest-additions CD image), si haces df -h podrás ver en la última línea el CD montado en un directorio llamado /media/VBOXADDITIONS..., hacer un cd a el y ejecutar el siguiente comando:

~~~shell
pkgadd -d VBoxSolarisAdditions.pkg
~~~

### INSTALAR SOLARIS DESKTOP:
Primeiro actualizamos, xa que pode dar problemas (--accept para aceptar licencias) 
~~~shell
pkg update --accept
~~~
~~~shell
pkg install solaris-desktop
~~~

Al terminar simplemente hacer reboot y ya ejecutará el escritorio.

### TWM:

Es sabido por todos que Yáñez odia gnome, entonces es por esto que vas a tener que hacer la siguiente configuración:

Crea un archivo llamado /usr/share/xsession/**twm.desktop** (cambiar en Exec y Tryexec __twm__ por __/opt/twm__) como se muestra:

![Captura de pantalla de 2023-03-28 11-14-06](https://user-images.githubusercontent.com/90336442/228189630-406cc136-e658-4fb0-b302-5ac92fa7edcb.png)

Ahora para tener el teclado en español:

~~~shell
echo "setxkbmap es" >> /opt/twm
echo "twm" >> /opt/twm
chmod +x /opt/twm
~~~
Por ultimo, para instalar los paquetes FSW, entras a firefox y los descargas. 
posteriormente:
~~~~shell
gunzip -c (nombre_archivo) | tar xvf -
pkgadd -d . (nombre_archivo)
~~~~

## OPENBSD (0_0)

con `xenodm` se inicia el login grafico, pero no se inicia por defecto asi que ponemos

~~~shell
rcctl enable xenodm
rcctl start xenodm
~~~

A continuación instalamos firefox y icewm:

~~~shell
pkg_add -v firefox
pkg_add -v icewm
echo exec icewm > .xsession
~~~

E instalamos mate (son todos los paquetes)

~~~shell
pkg_add -Iv nano
pkg_add -Iv mate-desktop mate-notification-daemon mate-terminal mate-panel 
pkg_add -Iv mate-session-manager mate-icon-theme mate-control-center mate-calc caja
~~~

una vez instalado toda la paquetería modificamos el `.xsession` (comentamos el arranque por icewm y ponemos el de mate)
Antes de nada imos meter ao usuario no grupo de usermod con 
~~~shell
usermod -G operator usuario
~~~
facemos reboot, e se facemos groups deberia aparecer o nome do noso ususario, proseguimos co nano a .xsession
~~~shell
#exec icewm
exec ck-launch-session mate-session
~~~

Metemoslle un reboot riquisimo e ao logear DEBERÍA cargar el mate (en caso contrario contactar a chuten mediante aspamientos)

> 		SI al abrir la terminal en MATE no se lee bien vais desde la terminal (menu de arriba) a Edit - Profiles - Edit - Colors  Y desmarcamos el themed y pones el  q quieras (blanco sobre negro por ejeplo) 

### Los ports del OpenBSD

Vale, por partes, primero de todo saber que versión de OpenBSD tenemos, para ello ejecutamos un `uname -a` y nos instalamos el wget con `pkg_add wget`

A continuación ejecutamos en función de la versión las siguientes llamadas (mi versión es la 7.2)

~~~shell
wget http://ftp.openbsd.org/pub/OpenBSD/7.2/ports.tar.gz
tar xzf ports.tar.gz -C /usr/
~~~

El descomprimir la movida lleva un cacho, no te agobies, acto seguido vamos a `cd /usr/ports/` dentro si hacemos `ls` veremos varios tipos de paquetes, los que nos interesan están en `www`, concretamente en

- `cd /usr/ports/www/links` 
- `cd /usr/ports/www/lynx`

Vamos a cada directorio y chantamos un `make install`, comprobamos que funciona y damos gracias al señor por estos sistemas operativos (si al darle a links le dais a enter y os deja escribir movidas, funciona)

## FREEBSD

![Captura de pantalla de 2023-03-27 18-17-33](https://user-images.githubusercontent.com/90336442/228002725-5c2593d1-17f4-4224-83cf-366f09d5e61b.png)

### AÑADIR AL USUARIO AL GROUP WHEEL:

No sé si soy el único, pero al instalar FreeBSD no podía hacer **su** con el usuario normal, esto se soluciona con el siguiente comando:

~~~shell
pw usermod sysadmin -G wheel
~~~

**DEBES AÑADIR EL USUARIO AL GROUP WHEEL**, sino no podrás ejecutar el mate.

### INSTALACIONES:

Para instalar las Guest Additions de virtualbox:

~~~shell
pkg update
pkg install virtualbox-ose-additions
~~~

Instalar finalmente:

~~~shell
pkg install xorg
pkg install mate-base mate-terminal
pkg install xdm
~~~

### CONFIGURACIONES:

Ahora mismo todo lo que has instalado no sirve para nada, así que empezemos por añadir las entradas que dijo Yáñez:

~~~shell
echo "vboxguest_enable=YES" >> /etc/rc.conf
echo "vboxservice_enable=YES" >> /etc/rc.conf
echo "dbus_enable=YES" >> /etc/rc.conf
echo "moused_enable=YES" >> /etc/rc.conf
~~~

**NO AÑADIR EL XDM TODAVÍA** ya que aún no está configurado.

Es hora de traer el xdm a /etc/rc.d para poder ejecutarlo en el arranque:

~~~shell
cp /usr/local/etc/rc.d/xdm /etc/rc.d/
~~~

Finalmente vamos a permitir que xdm arranque el mate, para ello crea en **/home/usuario** el fichero .xsession:

~~~shell
echo "setxkbmap -layout 'es'" >> .xsession
echo "exec mate-session" >> .xsession
~~~

**NOTA**: Puede que dbus os falle al arrancar, se debería evitar con el siguiente comando:

~~~shell
dbus-uuidgen --ensure
~~~

Para poder iniciar sesión con el root, debes copiar este fichero y pegarlo en **/root**

Y no olvidarse (__HAZ UN SNAPSHOT ANTES, ESTE PASO ES MUY PELIGROSO__):

~~~shell
echo "xdm_enable=YES" >> /etc/rc.conf
~~~

Ya tienes tu mate en FreeBSD

![Captura de pantalla de 2023-03-27 18-41-20](https://user-images.githubusercontent.com/90336442/228008513-11eb1f18-dcbd-4e7a-b0ee-6fe065a36803.png)

![Captura de pantalla de 2023-03-27 18-42-49](https://user-images.githubusercontent.com/90336442/228008836-4da9e420-958f-4fed-9927-e7e0b3d1b577.png)


Para instalar el __sistema de ports__:

~~~shell
portsnap fetch extract
cd /usr/ports/sysutils/neofetch && make install clean
cd /usr/ports/sysutils/htop && make install clean
~~~

## NETBSD

![Captura de pantalla de 2023-03-27 16-28-53](https://user-images.githubusercontent.com/90336442/227970937-df190cc4-5faa-4a3d-86b2-9bb099d6a3c8.png)

### **REQUISITO INDISPENSABLE: X11**

Para que el slim arranque, necesitamos tener el X11 instalado. X11 se puede instalar a través del sistema de ports (no lo conseguí) o bien en la instalación inicial del sistema si escogiste la opción. 

Para instalarlo, volver a insertar el cd de NetBSD y escoger la opción de instalar sets:

![Captura de pantalla de 2023-03-27 16-35-20](https://user-images.githubusercontent.com/90336442/227972993-bb513abf-cd5c-42ce-b853-fc18aeaf8b46.png)

Tras una serie de pasos, escoger custom installation:

![Captura de pantalla de 2023-03-27 16-36-43](https://user-images.githubusercontent.com/90336442/227973380-2994268f-dfbf-46aa-a732-0edac78458e1.png)

En X11 sets (el resto ponlo a NO porque se supone q ya están instalados), **instalar todos los sets**, yo no lo instalo porque ya lo tengo :):

![Captura de pantalla de 2023-03-27 16-39-07](https://user-images.githubusercontent.com/90336442/227974046-a3da96be-4144-4957-bc05-5b1173a49ecd.png)

Escoge la opción por HTTP, darle a la opción configure network (permitid la autoconfiguración) y luego a Get distribution para instalar X11:

![Captura de pantalla de 2023-03-27 16-46-52](https://user-images.githubusercontent.com/90336442/227976410-f4495ef3-f9f8-47c3-941e-67bd25997516.png)

Ya has instalado X11, escoge la opción de hacer reboot a la máquina y retira el disco:

**NOTA**:Si por algún casual te encuentras con problemas y necesitas reinstalar (no te preocupes, es rápido y no se desconfigura nada de refind ni de grub) deja el usuario que has creado, si no te deja logear con el usuario hacer lo siguiente en la terminal de rescate (al arrancar, escoger single-user mode):

- Este comando monta el / apropiadamente:
~~~shell
mount -va
~~~

- Cambiar la contraseña de usuario:
~~~shell
passwd usuario
~~~

- Listo:
~~~shell
reboot
~~~

### INSTALACIÓN DEL SISTEMA DE PORTS:

Utiliza estos dos comandos:

~~~shell
wget http://ftp.netbsd.org/pub/pkgsrc/stable/pkgsrc.tar.gz
tar -xzf pkgsrc.tar.gz -C /usr
~~~

### INSTALACIÓN DE SLIM Y DE MATE:

En NetBSD el sistema de paquetes es pkg_add, pero nos interesa utilizar **pkgin**, para instalarlo:

~~~shell
PKG_PATH="http://cdn.NetBSD.org/pub/pkgsrc/packages/NetBSD/$(uname -p)/$(uname -r|cut -f '1 2' -d.)/All/"
export PKG_PATH
pkg_add pkgin
~~~

Ahora para instalar slim:

~~~shell
pkgin install slim slim-themes
~~~

Si instalaste X11 como te dije no tendrás ningún problema.

Para instalar mate:

~~~shell
pkgin install mate
~~~

**NOTA**: puede que algunos paquetes no se hayan podido instalar, intenta correr el comando de nuevo y seguramente te saldrá un mensaje de poner **check_osabi=no** en pkg_install.conf, pues eso es lo que vas a hacer:

~~~shell
echo "CHECK_OSABI=no" >> /etc/pkg_install.conf
~~~

### CONFIGURACIÓN:

En este punto ya está todo instalado, pero no te va a funcionar nada. Empezemos con añadir los servicios al /etc/rc.d:

~~~shell
cp  /usr/pkg/share/examples/rc.d/slim /etc/rc.d
cp  /usr/pkg/share/examples/rc.d/dbus /etc/rc.d
~~~

Para poder arrancarlos, necesitamos añadir sus variables al /etc/rc.conf:

~~~shell
echo "dbus=YES" >> /etc/rc.conf
echo "slim=YES" >> /etc/rc.conf
~~~
Ahora mismo el slim está habilitado en el arranqué, **no reinicies todavía** puesto que slim aún no opera bien.

En este punto slim es **arrancable** pero no ejecutará ningún entorno de escritorio, para que slim sepa que debe ejecutar lo haremos por medio del archivo **.xinitrc**, en /home/usuaio:

~~~shell
echo "setxkbmap -layout 'es'" >> .xinitrc
echo "mate-session" >> .xinitrc
~~~

Si quieres logearte con root, debes copiar el mismo archivo al **/root**

Haz reboot y disfruta de tu nuevo NetBSD ;)

![Captura de pantalla de 2023-03-27 17-30-34](https://user-images.githubusercontent.com/90336442/227989441-63601303-63dc-47eb-8bba-257846b37257.png)

![Captura de pantalla de 2023-03-27 17-31-14](https://user-images.githubusercontent.com/90336442/227989604-79f3dbc2-d1a3-43f7-bbf8-60e315b05cc7.png)
