Title: Instalación de Fedora Server 28, parte 7, servidor de archivos SAMBA
Slug: fedora-server-28-parte-07
Summary: Crear una nueva partición lógica para compartir archivos por medio de SAMBA.
Tags: fedora, gnu linux, servidores, software libre
Date: 2018-06-24 22:17
Modified: 2018-06-24 22:17
Category: apuntes
Preview: fedora-logo-icon.png


Vamos a crear una partición dedicada a almacenar archivos llamada *biblioteca* y dentro de ella un directorio con *software* para todos los usuarios **SAMBA**.

### System Storage Manager

Instale el programa **System Storage Manager** para la administración de discos.

    # dnf install system-storage-manager

Verifique sus unidades y particionamiento actual...

    # ssm list

### Crear particiones

**System Storage Manager** nos brinda el comando **ssm** que es muy versátil; ya que en un comando se ordena...

* Crear una partición lógica con un tamaño de 128 GB.
* Hacerlo en el conjunto **LVM** que llamé *nogal* (vea la [parte 1]({filename}/apuntes/fedora-server-28-parte-01/fedora-server-28-parte-01.md)).
* Formatear con el sistema de archivos **ext4**
* Montar en el directorio `/mnt/biblioteca`

Cree el directorio y ejecute el comando **ssm**...

    # mkdir /mnt/biblioteca
    # ssm create -s 128G -p nogal -n biblioteca --fstype ext4 /mnt/biblioteca

Luego debe agregar una línea en `/etc/fstab` para que se monte la partición al encender el servidor...

    /dev/mapper/nogal-biblioteca  /mnt/biblioteca  ext4  defaults  1 2

Para poder copiar archivos cómodamente en `/mnt/biblioteca` otorgue permisos de escritura a su usuario **miusuario**. Por ejemplo...

    # mkdir /mnt/biblioteca/software
    # chown miusuario:users /mnt/biblioteca/software

### Instale y configure SAMBA en el servidor

Instale SAMBA...

    # dnf install samba

Edite...

    # nano /etc/samba/smb.conf

Lo más sencillo es configurar un *servidor solitario* con *software* para todos los usuarios y de sólo lectura. No deje de definir el nombre del grupo **MIGRUPO** y el nombre del servidor **MISERVIDOR**...

    [global]
    workgroup = MIGRUPO
    netbios name = MISERVIDOR
    security = user

    [software]
    comment = Biblioteca de software
    path = /mnt/biblioteca/software
    browseable = yes
    public = no
    writable = no

Agregue sus usuarios **SAMBA** y contraseñas cifradas con...

    # smbpasswd -a miusuario

Después de hacer cambios en `/etc/samba/smb.conf` haga una revisión con...

    # testparm

Configure el muro de fuego para que sólo desde la red local se pueda accesar...

    # firewall-cmd --permanent --zone=internal --add-service=samba

Arranque el *daemon*...

    # systemctl start smb.service

Recuerde que si cambia `/etc/samba/smb.conf` deberá reiniciar el *daemon*...

    # systemctl restart smb.service

Y para habilitar que inicie al encender, ejecute...

    # systemctl enable smb.service

### Quitar bloqueos de SELinux a SAMBA

Por defecto **SELinux** asegura el servidor limitando las comparticiones **SAMBA**. Para ver todos las *boleanos* que tienen que ver con samba ejecute...

    # getsebool -a | grep samba

Encontrará...

    samba_export_all_ro --> off
    samba_export_all_rw --> off

Cambie *con las consideraciones que esto implica* las *bandera* `samba_export_all_ro` que habilita a **SAMBA** a leer...

    # setsebool -P samba_export_all_ro on
    # getsebool -a | grep samba_export

Los *meta-permisos* en `/mnt/biblioteca/software` no tienen a `samba_share_t`...

    # ls -Z -l /mnt/biblioteca/software

Para que **SELinux** permita a **SAMBA** compartir *temporalmente*...

    # chcon -R -t samba_share_t /mnt/biblioteca/software
    # ls -Z -l /mnt/biblioteca/software

Haga permanentes los cambios en **SELinux** con...

    # semanage fcontext -a -t samba_share_t "/mnt/biblioteca/software(/.*)?"
    # restorecon /mnt/biblioteca/software
    # ls -Z -l /mnt/biblioteca/software

Más documentación en:

* [SELinuxProject.org - SambaRecipes](https://selinuxproject.org/page/SambaRecipes)

### Pruebas en los clientes

Puede exportar el usuario y contraseña a variables de entorno...

    $ export USER=miusuario
    $ export PASSWD=micontraseña

Liste los recursos compartidos con...

    # smbclient -L miservidor

Pruebe ingresar a un recurso compartido...

    $ smbclient //profeta/software

Si su administrador de archivos lo soporta, por ejemplo **Dolphin**, escriba esto en el campo de ubicación...

    smb://miservidor/

### Grupos de usuarios

Esto es para cuando necesita ofrecer un espacio en común para dos personas o más.

Igual, *con las consideraciones que esto implica* las *bandera* `samba_export_all_rw` que habilita a **SAMBA** a leer y escribir...

    # setsebool -P samba_export_all_rw on
    # getsebool -a | grep samba_export

Cree un nuevo directorio para su nuevo grupo de trabajo, por ejemplo, el grupo *ventas* con los usuarios *vendedor1* y *vendedor2*...

    # groupadd ventas
    # useradd vendedor1
    # useradd vendedor2
    # usermod -a -G ventas vendedor1
    # usermod -a -G ventas vendedor2
    # mkdir /mnt/biblioteca/ventas
    # chmod 0770 /mnt/biblioteca/ventas
    # chgrp ventas /mnt/biblioteca/ventas

Agregue en `/etc/samba/smb.conf`...

    [ventas]
    comment = Unidad de Sistemas de Infomacion
    path = /mnt/biblioteca/ventas
    valid users = @ventas
    write list = @ventas
    browsable = yes
    public = no
    writable = yes
    create mask = 0770
    Force create mode = 0770
    force group = ventas

Haga las autorizaciones de **SELinux**...

    # semanage fcontext -a -t samba_share_t "/mnt/biblioteca/ventas(/.*)?"
    # restorecon /mnt/biblioteca/ventas
    # ls -Z -l /mnt/biblioteca/ventas

### Más documentación

* [Fedora DOCS - Fedora System Administration Guide - Servers - File and Print Servers](https://docs.fedoraproject.org/f28/system-administrators-guide/servers/File_and_Print_Servers.html)
* [Setting Up Samba and Configure FirewallD and SELinux to Allow File Sharing on Linux/Windows Clients – Part 6](https://www.tecmint.com/setup-samba-file-sharing-for-linux-windows-clients/)

### Final

Le agradezco la lectura y aprovechamiento de estos apuntes; deseando que le sean de gran utilidad. Si tiene alguna sugerencia me la puede hacer llegar a mi correo electrónico.
