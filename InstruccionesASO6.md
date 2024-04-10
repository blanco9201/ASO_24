# GUIA CACHIPIRULI ASO 6

## TIP:
En virtualbox con una de las máquinas arrancadas ve a **Dispositivos->Portapapeles compartido->Bidireccional.**

Con esta configuración vas a poder **copiar y pegar comandos** de esta guía en la mayoría de los S.Os (para pegar en la terminal es **ctrl+shift+v**)

## OJO:
A la hora de editar los archivos PAM, poner las reglas en el fichero encima de donde se encuentren las reglas auth, tened en cuenta que su funcionamiento es parecido al de las ACLs, por lo que el orden importa. (Ver la sección **Entender PAM** al final de esta guía)

## Ubuntu

### Creación de los 500 usuarios
~~~shell
for num in `seq -w 0 499`; do useradd -m -p `mkpasswd qwerty$num` user$num; done
~~~
### Denegar root login en lightdm: 
Añadir en /etc/pam.d/lightdm:

`auth   required        pam_succeed_if.so user != root quiet_success`

No debería haber ningún problema en la colocación de esta regla, ya que el **required** revisa el resto de reglas que le siguen.

### Denegar root login en texto: 
Crear el fichero **/etc/securetty**:
~~~shell
touch /etc/securetty
~~~
El fichero contiene la lista de ttys en las que el root está permitido acceder (**man securetty**), si está vacío entonces no queremos que el root inicie sesión en ninguna tty.

Añadir en /etc/pam.d/login:

`auth  requisite  pam_securetty.so`

Se puede añadir como primera linea del fichero, ya que este módulo es ignorado por los usuarios no root (**man pam_securetty**) y en el caso del root queremos que siempre sea denegado.

### Para los usuarios user100 y user200 establecer la fecha de caducidad de la cuenta al 17 de abril de 2023.
~~~shell
usermod -e 2023-04-17 user100
usermod -e 2023-04-17 user200
~~~

**COMPROBACIÓN**: 
~~~shell
su user100
su user200
~~~
No te debería dejar entrar.

### Hacer que user001, user002 y user003 puedan administrar la máquina mediante sudo, igual que el usuario creado al instalar

~~~shell
usermod -aG sudo user001
usermod -aG sudo user002
usermod -aG sudo user003
~~~

### Sin crear un usuario nuevo Crear un directorio /var/INTERCAMBIO al que solo pueden acceder user008 y user009

Crear un nuevo grupo "intercambio":
~~~shell
groupadd intercambio
~~~

Asignarlo a user008 y 009:
~~~shell
usermod -aG intercambio user008
usermod -aG intercambio user009
~~~

Crear el directorio /var/INTERCAMBIO (desde el root):
~~~shell
mkdir /var/INTERCAMBIO
chgrp intercambio /var/INTERCAMBIO
chmod 2770 /var/INTERCAMBIO
~~~
El 2 del chmod quiere decir que se habilita el bit **setgid** (recordemos, tenemos los permisos para usuario, grupo y resto del mundo, pero antes también existen 3 campos, que son para activar el sticky bit, el **setgid** bit y el setuid bit en ese orden) que en este caso para un directorio permite que todos los ficheros creados en su interior pertenezcan al mismo grupo que el directorio (este comportamiento lo implementan los BSD por defecto (fuentes: trapas de teoría tema 4)).

### Hacer que puedan acceder otros usuarios conociendo un password

Poner password al grupo:
~~~shell
gpasswd intercambio
~~~

Para que el resto de usuarios puedan acceder al directorio, necesitan logearse en el grupo:
~~~shell
newgrp intercambio
~~~

### Hacer que user007 pueda cambiar dicho password:
~~~shell
gpasswd -A user007 intercambio
~~~
Este comando establece un administrador (user007) para el grupo intercambio, que le **permite cambiar la contraseña del grupo** y tambien la lista de miembros de dicho grupo, esta información se almacena en el **/etc/gshadow** (**man gshadow**).

Ahora user007 es el único que puede modificar el password del grupo:
~~~shell
gpasswd intercambio
~~~


## Devuan
Creamos un script para automatizar la creacion de 500 usuarios y le llamamos por ejemplo addusers.sh
```
#!/bin/bash

for i in {0..499}
do
    username=$(printf "user%03d" $i)
    password=$(printf "qwerty%03d" $i)

    useradd -m -p $(openssl passwd -1 $password) $username
done
```
Guardamos y ejecutamos en modo root.
```
bash addusers.sh
```
Podemos comprobar que estan creados utilizando el siguiente comando:
```
getent passwd
```

### (Otra manera) Creación de los 500 usuarios

~~~shell
for num in `seq -w 0 499`; do useradd -m -p `mkpasswd qwerty$num` user$num; done
~~~

### Denegar root login: en lightdm: 
Añadir en /etc/pam.d/lightdm:

`auth   required        pam_succeed_if.so user != root quiet_success`

No debería haber ningún problema en la colocación de esta regla, ya que el **required** revisa el resto de reglas que le siguen.

### Denegar root login en texto: 
Crear el fichero **/etc/securetty**:
~~~shell
touch /etc/securetty
~~~
El fichero contiene la lista de ttys en las que el root está permitido acceder (**man securetty**), si está vacío entonces no queremos que el root inicie sesión en ninguna tty.

Añadir en /etc/pam.d/login:

`auth  requisite  pam_securetty.so`

Se puede añadir como primera linea del fichero, ya que este módulo es ignorado por los usuarios no root (**man pam_securetty**) y en el caso del root queremos que siempre sea denegado.

### Para los usuarios user100 y user200 establecer la fecha de caducidad de la cuenta al 17 de abril de 2023.
~~~shell
usermod -e 2023-04-17 user100
usermod -e 2023-04-17 user200
~~~

### Hacer que user001, user002 y user003, junto con el usuario creado al instalar, puedan hacerse root mediante su sin necesidad de password. El resto de los usuarios no pueden hacerse root aunque conozcan el password:

Crear el grupo wheel:
~~~shell
groupadd wheel
usermod -aG wheel usuario
usermod -aG wheel user001
usermod -aG wheel user002
usermod -aG wheel user003
~~~

Comprobar:
~~~shell
groups usuario
~~~

Añadir en /etc/pam.d/su:

#Sólo los usuarios del grupo wheel pueden usar su:

`auth       required   pam_wheel.so root_only`

#Sólo los usuarios del grupo wheel pueden usar su sin password sólo para root:

`auth       sufficient pam_wheel.so trust root_only`


### Crear un alias (terminators) en /etc/sudoers formado por user004, user005, user006. Los miembros de este alias pueden apagar la máquina sin necesidad de autentificarse con su password


- https://www.digitalocean.com/community/tutorials/how-to-edit-the-sudoers-file-es


Edita el archivo /etc/sudoers con un editor de texto que tenga permisos de superusuario (hacemos `sudo visudo`).
Agregamos lo siguiente: 
~~~~shell
# alias TERMINATORS con el comando shutdown
Cmnd_Alias TERMINATORS = /sbin/shutdown 

%terminators ALL=(root) NOPASSWD: TERMINATORS
# todos los usuarios pertenecientes al grupo terminators (%terminators) pueden ejecutar el comando especificado(TERMINATORS) como superusuario ((root)) sin necesidad de introducir su contraseña (NOPASSWD:).
~~~~

Creamos y agregamos los usuarios al grupo terminators:

~~~~shell
sudo groupadd terminators

sudo usermod -aG terminators user004
sudo usermod -aG terminators user005
sudo usermod -aG terminators user006
~~~~

Para probarlo ejecutamos `sudo shutdown -h now` en distintos usuarios.

OTRA MANERA SIN CREAR GRUPOS SEGÚN LAS DIAPOS DEL PROFE (sí, sirven pa algo)
~~~~shell
# User alias specification
User_Alias TERMINATORS = user004, user005, user006
# Cmnd alias specification
Cmd_Alias POWERDOWN = /sbin/shutdown, /sbin/halt, /sbin/reboot
# User privilege specification
root    ALL=(ALL:ALL) ALL
TERMINATORS aso1=(root) NOPASSWD: POWERDOWN

~~~~


### Sin crear un usuario nuevo Crear un directorio /var/INTERCAMBIO al que solo pueden acceder user008 y user009

Crear un nuevo grupo "intercambio":
~~~shell
groupadd intercambio
~~~

Asignarlo a user008 y 009:
~~~shell
usermod -aG intercambio user008
usermod -aG intercambio user009
~~~

Crear el directorio /var/INTERCAMBIO (desde el root):
~~~shell
mkdir /var/INTERCAMBIO
chgrp intercambio /var/INTERCAMBIO
chmod 2770 /var/INTERCAMBIO
~~~
El 2 del chmod quiere decir que se habilita el bit **setgid** (recordemos, tenemos los permisos para usuario, grupo y resto del mundo, pero antes también existen 3 campos, que son para activar el sticky bit, el **setgid** bit y el setuid bit en ese orden) que en este caso para un directorio permite que todos los ficheros creados en su interior pertenezcan al mismo grupo que el directorio (este comportamiento lo implementan los BSD por defecto (fuentes: trapas de teoría tema 4)).

### Hacer que puedan acceder otros usuarios conociendo un password

Poner password al grupo:
~~~shell
gpasswd intercambio
~~~

Para que el resto de usuarios puedan acceder al directorio, necesitan logearse en el grupo:
~~~shell
newgrp intercambio
~~~

### Hacer que user007 pueda cambiar dicho password:
~~~shell
gpasswd -A user007 intercambio
~~~
Este comando establece un administrador (user007) para el grupo intercambio, que le **permite cambiar la contraseña del grupo** y tambien la lista de miembros de dicho grupo, esta información se almacena en el **/etc/gshadow** (**man gshadow**).

Ahora user007 es el único que puede modificar el password del grupo:
~~~shell
gpasswd intercambio
~~~
## OpenBSD

### Creación de los 500 usuarios:

Añadir el .xsession a /etc/skel:
~~~shell
cp /home/usuario/.xsession /etc/skel
~~~

Crear un script con el siguiente código:
~~~shell
for i in `seq -w 0 499`
do
useradd -m -p `encrypt qwerty$i` -k /etc/skel user$i
done
~~~

### Otra forma de hacerlo

Creamos un script para automatizar la creacion de 500 usuarios y le llamamos por ejemplo addusers.sh
```
#!/bin/ksh

for i in $(seq -w 0 499)
do
    username="user${i}"
    password="qwerty${i}"
    
    groupadd "$username"
    adduser -batch "$username" "$username" "$username" "$password"
done
```
Guardamos y le damos permisos y ejecutamos en modo root
```
chmod +x addusers.sh
./addusers.sh
```
### Para los usuarios user100 y user200 establecer la fecha de caducidad de la cuenta al 17 de abril de 2023.

Editar el /ect/master.passwd, ejecuta el comando:
~~~shell
vipw
~~~
Introducir en user100 y 200 la fecha del 17 de abril de 2023 en tiempo de unix en segundos (1681689600) en el 4º campo empezando por la derecha (con **man -s5 master.passwd** podemos ver las definiciones de cada campo del /etc/master.passwd (puede ser útil para el examen de prácticas)).
![Captura de pantalla de 2023-04-25 16-35-30](https://user-images.githubusercontent.com/90336442/234361746-8bc41e08-3f00-4916-83c0-37f4f3f03a88.png)

### En openBSD: crear una login class ’pringao’ en la que pondremos a user400. Para esta login class
- Tras un intento fallido de login la máquina empieza a poner delay entre los intentos de login
- El tamaño máximo de fichero es 1Mb
- El número mı́nimo de caracteres del password es 13
- No puede tener corriendo simultánemamente mas de 35 procesos
- Sus procesos se ejecutan con la mı́nima prioridad en el sistema

**Modificar /etc/login.conf con la nueva clase: (man login.conf para ver todos los posibles campos)**

![Captura de pantalla de 2023-04-27 20-16-55](https://user-images.githubusercontent.com/90336442/234964191-7975de9b-6954-48de-84fa-f41ed4a4ff38.png)

El umask es una "máscara de permisos", es decir, si yo pongo un umask de 022, lo que se hace es un 777-022=755 que son los permisos en octal por defecto en los nuevos archivos que vayamos a crear, aunque no se aplica a todos los comandos (si haces touch no los aplica, si haces mkdir si (**man umask**)).

**Asignar la clase:**

~~~shell
usermod -L pringao user400
~~~

**Probarlo:**
~~~shell
dd if=/dev/zero of=/home/user400/ficherode2megas bs=2048000 count=1
login user400
...
~~~

### NUEVO MÉTODO: Deshabilitar el login directo del root (tanto en modo texto como gráfico) en todos los S.O. (ojo: dejar que algun usuario pueda adquirir privilegios de administrador, bien mediante su o sudo)
Los que siguieron el anterior método (el de crear una clase llamada norootlogin) pueden prescindir de la loginclass para el root:

El comando **usermod -L clasenueva usuario** substituye la loginclass actual por "clasenueva" en el /etc/master.passwd, simplemente ponemos en blanco el campo: 
~~~shell
usermod -L "" root
~~~
Podemos ver como el campo **class** del siguiente comando está vacío, lo que nos indica que el root ya no tiene esa clase:
~~~shell
userinfo root
~~~
Una vez desasociada la clase al root, procedemos a editar el fichero **/etc/ttys**, eliminando la opción **secure** de las filas **ttyC**: 

![Captura de pantalla de 2023-05-27 13-03-21](https://github.com/FranciscoFuentesRamos/UnionSindicalTI/assets/90336442/422d67df-8460-4db1-8688-c17a4213201b)

La opción **secure** según el manual de openBSD (ver **man ttys**) nos dice que si está habilitada junto con la opción **on**, permite que los usuarios con el UID 0 (el root) puedan registrarse en la tty.

En openBSD las ttyC0, ttyC1... son a las que se pueden acceder con el clásico **ctrl+alt+Fx** siendo x el nº que corresponde con el nº de tty a la que accedes, en este caso como estamos en una máquina virtual, tenemos que pulsar **Host key (lo dice la vm abajo a la derecha, generalmente es el ctrl derecho) + Fx**, para volver a la terminal del entorno gráfico simplemente es probar con F1, sino F2...

**EJEMPLO**: si por ejemplo hemos echo **Host key + F2**, entonces habremos entrado en la **ttyC1**, si queremos volver a la tty del entorno gráfico seguramente sería la ttyC4, por lo que pulsamos **Host key + F5**.

## NETBSD:

### Creación de los 500 usuarios:
Añadir el .xinitrc a /etc/skel:
~~~shell
cp /home/usuario/.xinitrc /etc/skel
~~~

Crear el nuevo script:
~~~shell
for i in `seq -w 0 499`
do
useradd -m -p `pwhash qwerty$i` -k /etc/skel user$i
done
~~~

### Para los usuarios user100 y user200 establecer la fecha de caducidad de la cuenta al 17 de abril de 2023.

Editar el /ect/master.passwd, ejecuta el comando:
~~~shell
vipw
~~~
Introducir en user100 y 200 la fecha del 17 de abril de 2023 en tiempo de unix en segundos (1681689600) en el 4º campo empezando por la derecha.
![Captura de pantalla de 2023-04-25 16-35-30](https://user-images.githubusercontent.com/90336442/234361746-8bc41e08-3f00-4916-83c0-37f4f3f03a88.png)

### Hacer que user001, user002 y user003, junto con el usuario creado al instalar, puedan hacerse root mediante su sin necesidad de password. El resto de los usuarios no pueden hacerse root aunque conozcan el password

Meter a user001, user002 y user003 en el grupo wheel:

~~~shell
usermod -aG wheel user001
usermod -aG wheel user002
usermod -aG wheel user003
~~~

Añadir esta linea al /etc/pam.d/su para hacer "su" sin password:

`auth	sufficient	pam_group.so 	no_warn group=wheel root_only`

![Captura de pantalla de 2023-05-31 12-49-04](https://github.com/FranciscoFuentesRamos/UnionSindicalTI/assets/90336442/4f5e1558-633c-45c3-9ea2-aebf21685ca3)

En este caso la colocación de la regla es importante, ya que un **sufficient** otorga el acceso sin mirar el resto de módulos que le siguen (consultar la sección **Entender PAM** abajo de todo de esta guía).

### Deshabilitar el login directo del root (tanto en modo texto como gráfico) en todos los S.O. (ojo: dejar que algun usuario pueda adquirir privilegios de administrador, bien mediante su o sudo)
En **/etc/ttys** quitar la opción secure a todas las ttys en las que ponga **on**:

![Captura de pantalla de 2023-05-31 13-11-20](https://github.com/FranciscoFuentesRamos/UnionSindicalTI/assets/90336442/41eab478-825f-481d-a5d9-62be843388e7)

## Fedora
Para Fedora necesitamos hacer lo siguiente: 

### Creación de los 500 usuarios:
Primero creamos los 500 usuarios
~~~shell
for i in {000..499}
do
   useradd -m user$i -p $(mkpasswd qwerty$i) 
done
~~~
### Deshabilitamos el login directo del root
~~~shell
nano /etc/pam.d/lightdm
~~~
Añadimos esta línea
~~~shell
auth required pam_succeed_if.so user != root
~~~
~~~shell
nano /etc/pam.d/login
~~~
Añadimos esta línea
~~~shell
auth required pam_succeed_if.so user != root
~~~
### Hacer que user001, user002 y user003 puedan hacerse root sin necesidad de password:
~~~shell
nano /etc/pam.d/su
~~~
Añadimos estas dos líneas como sigue:

![Captura de pantalla de 2023-04-27 20-51-51](https://user-images.githubusercontent.com/90336442/234964027-291faa2c-bdba-4731-a0cc-98a31255d9eb.png)

Ahora el grupo
~~~shell 
usermod -aG wheel user001
usermod -aG wheel user002
usermod -aG wheel user003
usermod -aG wheel usuario
~~~  
### Usuarios user100 y user200 establecer la fecha de caducidad de la cuenta al 17 de abril de 2023
~~~shell
usermod -e 2023-04-17 user100
usermod -e 2023-04-17 user200
~~~
### Crear un directorio /var/INTERCAMBIO al que solo pueden acceder user008 y user009
~~~shell
groupadd intercambio
mkdir /var/INTERCAMBIO
chown root:intercambio /var/INTERCAMBIO
chmod 2770 /var/INTERCAMBIO
usermod -aG intercambio user008	
usermod -aG intercambio user009
~~~

### Hacer que puedan acceder otros usuarios conociendo un password:
Poner password al grupo:
~~~shell
gpasswd intercambio
~~~

Para que el resto de usuarios puedan acceder al directorio, necesitan logearse en el grupo:
~~~shell
newgrp intercambio
~~~

### Hacer que user007 pueda cambiar dicho password:
~~~shell
gpasswd -A user007 intercambio
~~~
Este comando establece un administrador (user007) para el grupo intercambio, que le **permite cambiar la contraseña del grupo** y tambien la lista de miembros de dicho grupo, esta información se almacena en el **/etc/gshadow** (**man gshadow**).

Ahora user007 es el único que puede modificar el password del grupo:
~~~shell
gpasswd intercambio
~~~

## FREEBSD:

### Crear los 500 usuarios:
Añadir el .xsession a /etc/skel:
~~~shell
cp /home/usuario/.xsession /etc/skel
~~~

Crear un script con el siguiente código:
~~~shell
for num in `seq -w 0 499`
do
#Crear usuario
pw user add -n user$num -m -s /bin/sh -k /usr/share/skel
#Establecer contraseña
echo "qwerty$num" | pw usermod -n user$num -h 0
#Si no es del grupo wheel no puede entrar en MATE (deben de ser las Guest Additions de VirtualBox)
pw usermod user$num -G wheel
done
~~~

### Deshabilitar el login directo del root (tanto en modo texto como gráfico) en todos los S.O. (ojo: dejar que algun usuario pueda adquirir privilegios de administrador, bien mediante su o sudo):

**IMPORTANTE**:Haz un snapshot antes de realizar el apartado

Crear un grupo para el root:
~~~shell
pw groupadd noroot
pw groupmod noroot -m root
~~~

Añadir en /etc/pam.d/xdm:

`auth 	requisite pam_group.so   deny group=noroot luser`

![Captura de pantalla de 2023-05-31 13-28-19](https://github.com/FranciscoFuentesRamos/UnionSindicalTI/assets/90336442/251e07e4-9082-48e1-9c63-9b0634f6a489)

Nuevamente es importante en dónde se coloca la regla por el **requisite**, porque si una regla no cumple con el criterio, se deniega el acceso independientemente del resto de reglas (ver el apartado **Entender PAM** al final de esta guía).

Añadir en /etc/pam.d/login:

`auth 	requisite pam_group.so   deny group=noroot luser`


### Para los usuarios user100 y user200 establecer la fecha de caducidad de la cuenta al 17 de abril de 2023.

Editar el /ect/master.passwd, ejecuta el comando:
~~~shell
vipw
~~~
Introducir en user100 y 200 la fecha del 17 de abril de 2023 en tiempo de unix en segundos (1681689600) en el 4º campo empezando por la derecha.
![Captura de pantalla de 2023-04-25 16-35-30](https://user-images.githubusercontent.com/90336442/234361746-8bc41e08-3f00-4916-83c0-37f4f3f03a88.png)

### Hacer que user001, user002 y user003, junto con el usuario creado al instalar, puedan hacerse root mediante su sin necesidad de password. El resto de los usuarios no pueden hacerse root aunque conozcan el password

Crear un grupo "sumaster":
~~~shell
pw groupadd sumaster
pw groupmod sumaster -m usuario
pw groupmod sumaster -m user001
pw groupmod sumaster -m user002
pw groupmod sumaster -m user003
~~~

Añadir las lineas al /etc/pam.d/su para hacer "su" sin password:

`auth            requisite       pam_group.so            no_warn group=sumaster root_only fail_safe ruser`

`auth            sufficient       pam_group.so            no_warn group=sumaster root_only fail_safe ruser`

### Hacer que cualquier usuario en el sistema pueda adquirir la identidad de user400:
En /etc/pam.d/su añadir:

`auth            sufficient      pam_group.so            no_warn group=user400 fail_safe luser`

![Captura de pantalla de 2023-05-31 13-29-47](https://github.com/FranciscoFuentesRamos/UnionSindicalTI/assets/90336442/99d18e48-11bb-4145-9936-f7083daa4a96)

De nuevo el orden de las reglas es importante.

## SOLARIS:

### Creación de los 500 usuarios:

Crear el archivo "users.txt":
~~~shell
for i in `seq -w 0 499`; do `echo "user$i qwerty$i" >> users.txt`;done
~~~

Bajarse el pass.expect de la web de ASO.

Crear el archivo "users.sh" con el siguiente contenido:
~~~shell
#!/bin/bash
IFS=$'\n'
contras=`cat /export/home/usuario/users.txt`
for i in $contras
do
a1=$(echo $i | awk '{print $1}')
useradd -m $a1
a2=$(echo $i | awk '{print $2}')
./pass.expect $a1 $a2
done
~~~

**NOTA**: Este script está hecho por alguien externo a este sindicato.

### Deshabilitar el login directo del root (tanto en modo texto como gráfico) en todos los S.O. (ojo: dejar que algun usuario pueda adquirir privilegios de administrador, bien mediante su o sudo)
Desde que el root es un rol (por lo tanto, no puede iniciar sesión) y no un usuario no hace falta hacer nada al respecto.

### Crear un rol (Instalador) que pueda instalar/eliminar software en el equipo, y otro (Reboteador) que pueda reiniciar el sistema. user100 puede asumir cualquiera de esos 2 roles, user200 el de Instalador y user300 solo el de Reboteador

**1º Crear el rol** (**man -k role -> man roleadd**)
~~~shell
roleadd -c "Instalar/eliminar software en el equipo" -s /usr/bin/pfbash -m -K profiles="Software Installation" Instalador
~~~
El profile de reiniciar **hay que crearlo**: (**man -k profile -> man prof_attr y man exec_attr**)

- Añadir a /etc/security/prof_attr
~~~shell
Reboteador:::Perfil para reboot: 
~~~

- Añadir a /etc/security/exec_attr
~~~shell
Reboteador:solaris:cmd:::/usr/sbin/reboot:uid=0
~~~

Ya creado el profile vamos con el role:
~~~shell
roleadd -c "Reiniciar la máquina" -s /usr/bin/pfbash -m -K profiles="Reboteador" Reboteador
~~~

### 2ºVerificamos consultando el archvio /etc/user_attr

### 3ºAsignar contraseña a los roles
~~~shell
passwd Instalador
passwd Reboteador
~~~
### 4ºCeder el derecho de iniciar sesión con estos roles
~~~shell
usermod -R +Instalador user100
usermod -R +Reboteador user100
usermod -R +Instalador user200
usermod -R +Reboteador user300
~~~
### 5ºprobar

Desde user100, user200 y user300:
~~~shell
su Instalador
su Reboteador
~~~
Se debe poder acceder desde:

- user100: a los 2
- user200: a Instalador
- user300: a Reboteador
El resto de usuarios **no pueden hacer su** a estos roles.

Instalador permite realizar operaciones con "pkg" y Reboteador ejecutar "reboot"

### Para los usuarios user100 y user200 establecer la fecha de caducidad de la cuenta al 17 de abril de 2023.

**NOTA**: Se recomienda realizar y comprobar que el apartado de los roles funciona

**Modifica el** <span style="color:red">/etc/shadow:</span>
~~~shell
chmod +w /etc/shadow
~~~
Modifica en user100 y en user200 el segundo campo empezando por la derecha e introduze el 17 de abril de 2023 en días de unix (19464):
![Captura de pantalla de 2023-04-25 19-41-35](https://user-images.githubusercontent.com/90336442/234367425-dfb8b6d6-963b-43f5-a7d5-84a70c4d6f1f.png)


**Por seguridad:**
~~~shell
chmod -w /etc/shadow
~~~

**COMPROBACIÓN**:
~~~shell
su user100
su user200
~~~

### Hacer que solo user001,user002 y user003, junto con el usuario creado al instalar puedad hacerse root

Agregar el rol de root:
~~~shell
usermod -R +root user001
usermod -R +root user002
usermod -R +root user003
~~~

Comprobar:
~~~shell
cat /etc/auth_attr
~~~


### Sin crear un usuario nuevo Crear un directorio /var/INTERCAMBIO al que solo pueden acceder user008 y user009

Crear un nuevo grupo "intercambio":
~~~shell
groupadd intercambio
~~~

Asignarlo a user008 y 009:
~~~shell
usermod -G +intercambio user008
usermod -G +intercambio user009
~~~

Crear el directorio /var/INTERCAMBIO (desde el root):
~~~shell
mkdir /var/INTERCAMBIO
chgrp intercambio /var/INTERCAMBIO
chmod 2770 /var/INTERCAMBIO
~~~

### Hacer que puedan acceder otros usuarios conociendo un password

Copia cualquier contraseña hasheada del /etc/shadow y añádela al /etc/group (ten en cuenta que la contraseña que copies será la del grupo):

~~~shell
nano /etc/shadow
~~~

Selecciona el hash y dentro de nano haz **Ctrl + u**


En /etc/group en el segundo campo empezando por la izquierda de intercambio, pega el hash dentro de nano con **shift + insert**:
~~~shell
nano /etc/group
~~~

![Captura de pantalla de 2023-04-26 18-37-21](https://user-images.githubusercontent.com/90336442/234646775-b89fad9d-1594-403c-a88b-9c052c509e24.png)

Para comprobar la consistencia del /etc/group:
~~~shell
grpck
~~~

Para que el resto de usuarios puedan acceder al directorio, necesitan logearse en el grupo:
~~~shell
newgrp intercambio
~~~

## Entender PAM:
Como ya se dijo, las PAM son como las ACLs, una lista secuencial de reglas que, si una coincide, ya se deniega o permite el acceso sin importar el resto de reglas (excepto si se usa required). Vamos a verlo con un ejemplo de **linux**:

Esta regla significa que si no eres del grupo verde no te puedes hacer root: 

`auth requisite pam_wheel.so group=verde root_only trust use_uid`

Esta regla permite a los del grupo azul hacer su a root sin password: 

`auth sufficient pam_wheel.so group=azul root_only trust use_uid`

Como digo es una lista secuencial, es decir, que si ponemos en el fichero estas 2 reglas en este orden, estamos permitiendo hacer su a root sin password a los usuarios que pertenecen al grupo verde y al grupo azul. Si revertimos el orden de las reglas, entonces se pueden hacer su a root sin password los que pertenezcan al grupo azul (y los que no pertenezcan al grupo azul, si tampoco pertenecen al grupo verde se les deniega directamente el acceso).

`auth requisite pam_wheel.so group=verde root_only trust use_uid`

`auth required pam_wheel.so group=azul root_only trust use_uid`

`auth requisite  pam_wheel.so  deny group=violet root_only trust use_uid`

En este caso, el **required** funciona como un requisite (si no se cumple la condición no se garantiza el acceso) y en el caso de que se cumpla la condición se miran las siguientes reglas, si hay una regla requisite con la que coincida, se deniega el acceso, en caso contrario se acepta (osea, eres capaz de autenticarte con password). **Siempre vas a tener que introducir la contraseña si usas un required** (aunque más abajo haya un sufficient).

En este caso:
- Perteneces al grupo verde, azul y violet: acceso denegado (te pide el password y aunque aciertes no puedes entrar)
- Perteneces al grupo verde y azul: acceso concedido (te pide el password y si aciertas puedes entrar)

`auth requisite pam_wheel.so group=verde root_only trust use_uid`

`auth sufficient pam_wheel.so group=azul root_only trust use_uid`

`auth requisite  pam_wheel.so  deny group=violet root_only trust use_uid`

En este caso:
- Perteneces al grupo verde, azul y violet: acceso concedido (entras sin password)
- Perteneces al grupo verde y azul: acceso concedido (entras sin password)


Por último destacar 2 cosas:

- Las opciones de un módulo son muy importantes, en el caso del pam_wheel.so, la regla root_only hace que solo se mire si haces su a root, por ejemplo esta regla: `auth sufficient pam_wheel.so group=azul trust use_uid` permite hacer su sin password **a todas las cuentas del sistema** si se pertenece al grupo azul.
- 
- Los módulos entre S.Os cambian, pam_wheel en linux hace lo mismo que pam_group en FreeBSD. Tampoco todos tienen las mismas opciones, pam_group en FreeBSD dispone de las opciones "ruser" y "luser" mientras que el pam_group de NetBSD, que hace lo mismo, no tiene estas opciones. Hay muchos de estos casos e intentar aprenderse todos los módulos de todos los S.Os resulta **casi imposible**. 

En resumen, es vital mirar el manual del sistema operativo en busca de los módulos que podemos usar (**man -k pam | grep 8 (en Solaris grep 5)**), las descripciones de lo que hacen los módulos y sus opciones suelen ser sencillas, por lo que aprender a usar un módulo de un S.O mirando el manual es sencillo.
