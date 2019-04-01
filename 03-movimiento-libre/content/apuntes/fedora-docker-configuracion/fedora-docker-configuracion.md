Title: Configuración de Docker en Fedora
Slug: fedora-docker-configuracion
Summary: Listo los comandos para configurar Docker en Fedora con su partición dedicada y control por el usuario.
Tags: fedora, docker
Date: 2019-01-24 10:10
Modified: 2019-01-24 10:30
Category: apuntes
Preview: preview.png
Image: imagen.png


### Requisitos

Este apunte es para Fedora Linux. Su objetivo es usar una partición dedicada a Docker y que sea controlable a través de una cuenta de usuario.

Ya debe tiene instalado...

* LibVirt
* System Storage Manager

Por defecto, Fedora Linux usa LVM para admnistrar el disco duro; se tiene un volumen con espacio libre, más de 64 GB.

### Partición dedicada

Es buena idea crear una partición dedicada. Montada en una ruta temporal...

    # mkdir /mnt/docker
    # ssm create -s 64G -p integridad -n docker --fstype ext4 /mnt/docker

Cambie la etiqueta de la unidad...

    # e2label /dev/mapper/integridad-docker Docker

Copie todo el contenido que se ha puesto en la instalación al directorio de montaje temporal...

    # cd /var/lib/docker/
    # tar cvf - * | (cd /mnt/docker/; tar xf -)

Desmonte del directorio temporal...

    # umount /mnt/docker

Para que se monte al encender, edite...

    # nano /etc/fstab

Y agregue esta línea...

    /dev/mapper/integridad-docker  /var/lib/docker  ext4  defaults  1 2

Monte en el directorio definitivo...

    # mount /var/lib/docker

Re-aplique los permisos SELinux...

    # docker-storage-setup
    # restorecon -vR /var/lib/docker/

Reinicie el sistema...

    # systemctl reboot

Verifique que se haya iniciado con éxito el daemon...

    # systemctl status docker

Ya no se necesita el directorio temporal...

    # rmdir /mnt/docker

### Control por usuario

Revise si existe el grupo docker...

    # grep docker /etc/group

De NO existir, habrá que crearlo con...

    # groupadd docker

Agregue su usuario al grupo...

    # gpasswd -a guillermovaldes docker

Reinicie el daemon...

    # systemctl restart docker

Cierre la sesión y vuelva a entrar.
