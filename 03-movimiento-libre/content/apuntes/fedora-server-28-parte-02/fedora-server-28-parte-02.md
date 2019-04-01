Title: Instalación de Fedora Server 28, parte 2, configuración básica de red
Slug: fedora-server-28-parte-02
Summary: Proseguimos con la configuración de OpenSSH y de los dispositivos de red.
Tags: fedora, gnu linux, servidores, software libre
Date: 2018-06-24 22:12
Modified: 2018-06-24 22:12
Category: apuntes
Preview: fedora-logo-icon.png


### Detalles de los dispositivos de red

Liste todos los dispositivos de red con...

    # lspci | grep -i ethernet

Averigue detalles, como la velocidad actual de la conexión con...

    # ethtool enp1s0
    # ethtool enp4s0

También puede instalar **lshw**.

    # dnf install lshw

Y ejecutar...

    # lshw -class network

### OpenSSH para usar par de llaves

Copie la llave pública **desde su equipo personal** al servidor con el comando...

    $ ssh-copy-id -i /home/usuario/.ssh/id_rsa.pub usuario@192.168.n.m

Por supesto, cambie `usuario` por su nombre de usuario y `192.168.0.n.m` por la dirección IP del nuevo servidor.

Luego edite de nuevo `/etc/ssh/sshd_config` y cambie...

    LoginGraceTime 2m
    PermitRootLogin no
    StrictModes yes
    MaxAuthTries 8
    MaxSessions 4
    PubkeyAuthentication yes
    PermitEmptyPasswords no
    PasswordAuthentication no

**Advertencia:** `PasswordAuthentication no` deshabilita el acceso con usuario y contraseña.

Reinicie el *daemon* para aplicar estos cambios.

    # systemctl restart sshd

### Desactivar Network Manager

Por defecto, **Network Manager** está activo, por lo que los dispositivos de red son configurados por sus operaciones.

    # systemctl list-units --type=service
    # systemctl status NetworkManager

Y el servicio `systemd-networkd` está deshabilitado.

    # systemctl status systemd-networkd

Tome nota de los dispositivos de red...

    # ip addr

E identifique cuál está conectado con la red local y cuál será para Internet. Por ejemplo:

* enp4s0 estará conectado al WAN (es decir, al Internet) con la dirección IP 192.168.1.21/24 (elija cualquiera que no se use)
* enp1s0 estará conectado a la LAN (red local) con la dirección IP 192.168.4.1/24

Para configurar los dispositivos de red de forma manual cámbiese al directorio...

    # cd /etc/systemd/network/

Cree el archivo de configuración...

    # nano enp4s0.network

Con este contenido que define su dirección IP, *puerta de acceso*, servidores DNS e *ip forward*...

    [Match]
    Name=enp4s0

    [Network]
    Address=192.168.1.21/24
    Gateway=192.168.1.254
    DNS=8.8.8.8
    DNS=8.8.4.4
    IPForward=yes

**Nota:** Usé los [DNS públicos de Google](https://developers.google.com/speed/public-dns/). Elija los de su preferencia.

Cree el archivo de configuración...

    # nano enp1s0.network

Con este contenido...

    [Match]
    Name=enp1s0

    [Network]
    Address=192.168.4.1/24
    IPForward=yes

Ahora deshabilite **Network Manager** y habilite **systemd-networkd**

    # systemctl disable NetworkManager
    # systemctl disable ModemManager
    # systemctl mask NetworkManager
    # systemctl mask NetworkManager-wait-online
    # systemctl enable systemd-networkd

### Apague el servidor

    # systemctl poweroff

Estando apagado los cables de red del modem a `enp4s0` y de `enp1s0` a su *switch* o *access point*.

### Encienda el servidor

Revise con con el comando `networkctl`...

    # networkctl
    IDX LINK             TYPE               OPERATIONAL SETUP
      1 lo               loopback           carrier     unmanaged
      2 enp4s0           ether              routable    configured
      3 enp1s0           ether              routable    configured

También revise...

    # ip addr show enp1s0
    # ip addr show enp4s0
    # route -n

### Continuación...

[Vaya a la parte 3, servicios DHCP y DNS]({filename}/apuntes/fedora-server-28-parte-03/fedora-server-28-parte-03.md).
