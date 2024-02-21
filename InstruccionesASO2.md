# Guía Chachipiruli ASO 2

Iniciamos la wecha esta creando un disco de 16 GB al cual luego más adelante le meteremos los otros 16, (Habilitar el EFI y en red escoger el primer cable Intel q aparezca).

Una vez creado y configurado la red, abrimos de nuevo la configuracion y en Almacenamiento, le damos a crear disco

![Captura desde 2023-03-15 11-38-31](https://user-images.githubusercontent.com/91732612/225305583-c39ff12a-9190-4760-b886-059eab5e2006.png)


ahi creamos otro disco con 16 GB, y luego lo seleccionamos el nuevo volumen para que aparezca "attached".

![Captura desde 2023-03-15 11-39-46](https://user-images.githubusercontent.com/91732612/225305648-2da14d65-6deb-4eb1-a0ac-428518ce6a98.png)

## Ubuntu Server

Las movidas de red se skipean

- Usamos el Disco entero, tal cual viene marcado por defecto, pero desactivamos el "Set up this disk as an LVM group"
- Borramos el / para preasignado para reasignar las movidas manualmente desde el freespace añadiendo las GPT.

A partir de este punto se skipea todo.

### Grub Ubuntu.

Editamos el grub por defecto con `nano /etc/default/grub` y cambiamos el `GRUB_TIMEOUT_STYLE=hidden` a `GRUB_TIMEOUT_STYLE=menu` y le ponemos un timeout razonable

chantamos un update-grub, comprobamos que funciona y nos comemos unas chocobon por el trabajo bien hecho :D

## NETBSD

Usamos un volumen fisico y si lo hiciste como yo usarás el wd1, en el hacemos la partición segundo nos indique el magnánimo yañez. Continuamos por defecto y dejamos instalando.

Configuramos la passwd del root y el usuario y piola todo.

### Volvemos al Ubuntu grub

(con df -h vemos donde esta el / y ponemos eso en el /dev/sdaX)

vamos a `nano /etc/grub.d/40_custom` y metemos 3 entradas:

![Captura desde 2023-03-15 12-39-36](https://user-images.githubusercontent.com/91732612/225306494-5b5e6a31-189c-4c95-9f40-4015714c012b.png)


A continuación vamos a `/boot/efi/EFI` y ahi creamos una carpeta llamada **KERNELS** como decimos en el grub anterior y vamos atrás al /boot y de ahi copiamos:

~~~shell
cp vmlinuz-5.15.0-67-generic efi/EFI/KERNELS/cpkernel
cp initrd.img-5.15.0-67-generic efi/EFI/KERNELS/cpinitrd
~~~

Hacemos update-grub y reboteamos, pasamos a netbsd

### Volvemos a NetBSD

Al cargar la entrada del grub va a decirte uwu y ponemos consecutivamente 

~~~shell
dk5
dk6
ffs
[y un enter]
~~~

### WARNING Esto no es necesario:
Al realizar la práctica, pensaba que no podías ejecutar el bootx64.efi de NetBSD sin tener que montarlo antes manualmente, por lo que no es necesario realizar estos pasos, salta directamente a la parte del REFIND.

En terminal escribimos `mount` y en principio debería faltarte la partición EFi en la dk4, procedemos:

~~~
cd /
mkdir -p /boot/efi
~~~

Antes de nada hacemos un:

~~~shell
dmesg | grep wd1
~~~

Y nos interesa la info del dk4 (dónde esté instalada la partición EFI, de type=msdos)

![Captura desde 2023-03-15 12-25-04](https://user-images.githubusercontent.com/91732612/225305752-e3e6f23f-1f67-4fc8-b3bf-b5aa5efbdf90.png)


Por desgracia tendremos que usar el vi en el /etc/fstab y añadir una linea NAME con el codigo de antes y ponerle lo que continúa

![Captura desde 2023-03-15 12-29-37](https://user-images.githubusercontent.com/91732612/225305774-9bdae9cb-f59b-48c2-ac9e-019141f481d7.png)


Una vez guardado y pasado el calvario del VI, escribimos

~~~shell
mount /dev/dk4 /boot/efi
~~~

Para comprobar si funciona debería existir el bootx64.efi en el `/boot/efi/efi/boot/`.

## REFIND (UBUNTU)

~~~shell
sudo apt-get install refind
~~~

(se le de a YES)

Apuntar el resultado de la consulta en 

~~~shell
blkid
~~~

De ahi, donde estea el kernel (en micaso en sda4), copiamos todos los numericos

![Captura desde 2023-03-15 12-54-47](https://user-images.githubusercontent.com/91732612/225306607-47fc057f-e722-4749-8cd9-d5c93caa2df9.png)

y hacemos un `nano /boot/efi/EFI/refind/refind.conf`

Hai q baixar moito.

Dentro buscamos el menuentry de UBUNTU y le borramos el "disable" y le añadimos cosas

![Captura desde 2023-03-15 13-04-32](https://user-images.githubusercontent.com/91732612/225306167-30ae0a45-40ec-4062-8252-3130869ff5a3.png)


- El volume de ubuntu es el PARTUUID
- el options en las 2 submenuentry es la UUID del  ubuntu
- Y mas de lo mismo con la anterior UUID de netbsd sacada anteriormente

Acto seguido subimos muy mucho y descomentamos la linea de dont_scan_dirs y añadimos al final ,EFI/KERNELS

![Captura desde 2023-03-15 13-01-21](https://user-images.githubusercontent.com/91732612/225306109-142b7548-bc21-4666-a495-4fa6205f7799.png)

Dentro del menu, encima del icono para mostrar las submenuentry pulsamos TAB, las menuentry se prueban pulsando ENTER y para suprimir una entrada q sobre le damos a suprimir encima de la entrada y confirmamos :D

![Captura desde 2023-03-15 13-07-52](https://user-images.githubusercontent.com/91732612/225306030-788f5533-0b32-4d14-af09-a0b706d11e36.png)

prueba que todas las entradas funcionen y ya.



-Extra:

Para el apartado de la configuración del Refind, podemos simplificar un poco el proceso, en vez de depender del identificador del volumen utilizamos el siguiente comando para poder cambiar la etiqueta del volumen:
~~~shell
e2label /dev/sda2 NETBSD
~~~

Nos quedaría algo de este estilo:

![image](https://user-images.githubusercontent.com/90767186/226745776-182359bd-6df1-4374-883d-498ba7e0aeea.png)



