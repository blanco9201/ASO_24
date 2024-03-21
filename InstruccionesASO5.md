# GUIA CACHIPIRULI ASO 5

## FreeBSD Jails

- Mais info:
    - [Enlace1](https://freebsdfoundation.org/freebsd-project/resources/introduction-to-freebsd-jails/)
    - [Enlace2](https://docs.freebsd.org/en/books/arch-handbook/jail/)

- Creamos carpeta para a jail `mkdir -p /usr/jail/jaulita` e `cd /usr/jail/jaulita`
- Instalamos wget y: `wget https://ftp.freebsd.org/pub/FreeBSD/releases/amd64/amd64/13.1-RELEASE/base.txz`
- Instalar también (no recomendado) los ports: `wget https://ftp.freebsd.org/pub/FreeBSD/releases/amd64/amd64/13.1-RELEASE/ports.txz`
- Descomprimimos o contido na carpeta jaulita:
    - `tar xvJf /home/usuario/base.txz`(OCUPA 1GB)
    - `tar xvJf /home/usuario/ports.txz`(OCUPA 1GB)

- Ahora configuramos o jail. En `/etc/jail.conf`:
```
jaulita {
    path = /usr/jail/jaulita;
    mount.devfs;
    host.hostname = jailcilla;
    ip4.addr = 10.0.2.25;
    interface = em0;
    exec.start = "/bin/sh /etc/rc";
    exec.stop = "/bin/sh /etc/rc.shutdown";
}
```

- Para facer que o jail se inicie en boot poñemos `jail_enable="YES"` en /etc/rc.conf.
- Ahora iniciamos o Jail `jail -c jaulita` e comprobamos o numero de jail con `jls`. Como solo temos un jail o JID sera 1.

- Abrimos un shell dentro do jail con `jexec 1 tcsh`.

- Unha vez estamos dentro do jail:
    - Creamos os usuarios (elbueno,elfeo,elmalo) con `adduser`.
    - Poñemos password ao root no jail. `passwd root`.
    - Comprobamos os dos usuarios. Desde fora do jail executar `ls -lah /usr/jail/jaulita/home/`
        - Si deixamos todo por defecto ao crear os usuarios vemos que o primeiro usuario colle o UID do usuario por defecto 1001 (usuario) e o resto vai incremental (1002 e 1003).
        ![FreeBSD_Permisos1](resources/jail_permisos_bsd.png)
        - Si non queremos que pase eso cando creamos o primeiro usuario decimoslle que colla UID 1002.
        ![FreeBSD_Permisos2](resources/jail_permisos_2.png)


## LXC Containers

- Mais info:
    - [Enlace1](https://ubuntu.com/server/docs/containers-lxc)

- Instalamos o paquete LXC `sudo apt install lxc`.

- Creamos o contenedor `lxc-create -t ubuntu -n container`

- Listamos os contenedores con `lxc-ls -f`
- Iniciamos o contenedor con `lxc-start -n container`
- Para que se inicie automaticmente `lxc-autostart -n container`
- Metemonos dentro do contenedor con `lxc-attach -n container`
    - Poñemos password ao root no container. `passwd root`.
    - Creamos os usuarios (elbueno,elfeo,elmalo) con `adduser`.
    - Comprobamos os dos usuarios. Desde fora do container executar `ls -lah /var/lib/lxc/container/rootfs/home`
        - Aqui por defecto no contenedor xa ven o usuario ubuntu creado. E ao crear o resto de usuarios empezan en 1002....
        ![LXC](resources/lxc.png)
       

   

## Solaris Zones

- Empezamos a configurar zona:
    - Creamos file system: `zfs create rpool/zonita`
    - Entramos con `zonecfg -z zonita`:
        - `create`
        - `set zonepath=/rpool/zonita`
        - `set autoboot=true`
        - `set bootargs="-m verbose"`
        - `verify`
        - `commit`
        - `exit`
    - Listamos as zonas con: `zoneadm list -icv`
    - Instalamos a zona con `zoneadm -z zonita install`
    - Booteamos a zona con `zoneadm -z zonita boot`
    - Entramos na zona `zlogin -C zonita`
        - Creamos usuarios con `useradd -d dir -m username`
        - Use the `–d localhost:/export/home/username`
        - Listamos con `ls -lah /export/home/`
        ![Solaris](resources/solaris.png)




