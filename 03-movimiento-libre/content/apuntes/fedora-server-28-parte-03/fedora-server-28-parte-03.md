Title: Instalación de Fedora Server 28, parte 3, servicios DHCP y DNS
Slug: fedora-server-28-parte-03
Summary: Configuraremos el servidor para que proporcione direcciones IP y sea el DNS de la red local con DNSmasq.
Tags: fedora, gnu linux, servidores, software libre
Date: 2018-06-24 22:13
Modified: 2018-06-24 22:13
Category: apuntes
Preview: fedora-logo-icon.png


### DNSmasq

Instale DNSmasq...

    # dnf install dnsmasq

Cree un archivo para los DNS externos...

    # nano /etc/dnsmasq-resolv.conf

Con los servidores DNS que elija (en este ejemplo están activos los de Google)...

    # Cloudflare
    #nameserver 1.1.1.1
    #nameserver 1.0.0.1

    # Google
    nameserver 8.8.8.8
    nameserver 8.8.4.4

    # OpenDNS
    #nameserver 208.67.222.222
    #nameserver 208.67.220.220

Edite la configuración de DNSmasq...

    # cd /etc
    # mv dnsmasq.conf dnsmasq.conf.original
    # nano dnsmasq.conf

Ajuste el contenido de `/etc/dnsmasq.conf` de acuerdo a sus necesidades.
Cambie el nombre de dominio `miredlocal.lan` y los nombres de los equipos.
También las direcciones IP que son `192.168.4.n` desde 101 a 199.

    # Set the DHCP server to authoritative mode. In this mode it will barge in
    # and take over the lease for any client which broadcasts on the network,
    # whether it has a record of the lease or not.
    dhcp-authoritative

    # Never forward plain names (without a dot or domain part)
    domain-needed

    # Los nombres sencillos en /etc/hosts seran extendidos con el nombre de dominio completo
    expand-hosts

    # Dispositivos de red que ademas de localhost estan en escucha
    interface=enp1s0

    # Dominio
    domain=miredlocal.lan

    # Rango de direcciones IP y duracion
    dhcp-range=192.168.4.101,192.168.4.199,1h

    # Usuario y grupo a usar por el daemon
    user=dnsmasq
    group=dnsmasq

    # Dispositivos con direcciones IP fijas
    #dhcp-host=hh:hh:hh:hh:hh:hh,192.168.4.11  # equipo1
    #dhcp-host=hh:hh:hh:hh:hh:hh,192.168.4.12  # equipo2
    #dhcp-host=hh:hh:hh:hh:hh:hh,192.168.4.13  # equipo3
    #dhcp-host=hh:hh:hh:hh:hh:hh,192.168.4.14  # equipo4

    # Archivo donde se configuran los servidores DNS externos
    resolv-file=/etc/dnsmasq-resolv.conf

    # Ofrecerce como servidor de tiempo
    #dhcp-option=42,0.0.0.0

    # URL del script de configuración automática de proxy en el navegador
    #dhcp-option=252,http://192.168.4.1/proxy.pac
    # En cambio, para Windows, cuando no se tiene, se envía un avance de línea
    #dhcp-option=252,"\n"

    # Enviar el log a un archivo
    log-facility=/var/log/dnsmasq.log

    # Agregar a la bitácora cada consulta
    #log-queries

    # Set the cachesize
    cache-size=256

    # Include all files in /etc/dnsmasq.d except RPM backup files
    conf-dir=/etc/dnsmasq.d,.rpmnew,.rpmsave,.rpmorig

Si va usar direcciones IP fijas por mac address, configure `/etc/hosts` como en este ejemplo...

    192.168.4.1   proxy wpad
    192.168.4.11  equipo1
    192.168.4.12  equipo2
    192.168.4.13  equipo3
    192.168.4.14  equipo4

Y cambie el contenido de `/etc/resolv.conf` a...

    search miredlocal.lan
    nameserver 127.0.0.1

Antes de arrancar el daemon, vamos otorgar al grupo `root` permisos de escritura en `/var/lib/dnsmasq`...

    # chgrp root /var/lib/dnsmasq
    # chmod g+w /var/lib/dnsmasq

Arranque y habilite

    # systemctl start dnsmasq
    # systemctl enable dnsmasq

También habilite Network Name Resolution

    # systemctl start systemd-resolved
    # systemctl enable systemd-resolved

### Configuración temporal del muro de fuego

En este punto del proceso, el muro de fuego no permite conexiones a los servicios de DNS y DHCP...

    # firewall-cmd --list-services
    ssh dhcpv6-client cockpit

Para activar el servicio de Internet vamos a habilitar en *enmascarado de redes*...

    # firewall-cmd --add-masquerade

Y abrir la escucha de los puertos dedicados al DNS y DHCP...

    # firewall-cmd --add-service=dns --add-service=dhcp

Verifique con...

    # firewall-cmd --list-services
    ssh dhcpv6-client cockpit dns dhcp

Cabe tener en cuenta que los cambios en el muro de fuego se perderán al apagar o reiniciar el servidor. A menos que haga estos cambios permanentes con el comando...

    # firewall-cmd --runtime-to-permanent

### Logrotate

Para evitar que la bitácora de DNSmasq sea un archivo enorme cree esta regla de logrotate:

    # nano /etc/logrotate.d/dnsmasq

Con este contenido...

    /var/log/dnsmasq.log {
        daily
        notifempty
        missingok
        sharedscripts
        postrotate
            [ ! -f /var/run/dnsmasq.pid ] || kill -USR2 `cat /var/run/dnsmasq.pid`
        endscript
        create 0640 dnsmasq root
    }

### Continuación...

[Vaya a la parte 4, muro de fuego para ruteador]({filename}/apuntes/fedora-server-28-parte-04/fedora-server-28-parte-04.md).
