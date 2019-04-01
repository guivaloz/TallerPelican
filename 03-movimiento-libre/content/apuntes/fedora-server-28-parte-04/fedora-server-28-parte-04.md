Title: Instalación de Fedora Server 28, parte 4, muro de fuego para ruteador
Slug: fedora-server-28-parte-04
Summary: Configuraremos el muro de fuego para que los servicios de la red local sólo estén abiertos en el dispositivo correspondiente.
Tags: fedora, gnu linux, servidores, software libre
Date: 2018-06-24 22:14
Modified: 2018-06-24 22:14
Category: apuntes
Preview: fedora-logo-icon.png


### Asigne una zona a cada dispositivo de red

Recordemos que en estos ejemplos...

* enp4s0 está conectado al WAN (es decir, al Internet)
* enp1s0 está conectado a la LAN (red local)

Por lo que asignameros una zona a cada dispositivo...

* enp4s0 en external
* enp1s0 en internal

Ejecutando estos comandos...

    # firewall-cmd --set-default-zone=internal
    # firewall-cmd --zone=external --add-interface=enp4s0
    # firewall-cmd --zone=internal --add-interface=enp1s0

Verifique...

    # firewall-cmd --get-active-zones
    external
      interfaces: enp4s0
    internal
      interfaces: enp1s0

Otra forma de verificar es obtener la zona por dispositivo...

    # firewall-cmd --get-zone-of-interface=enp4s0
    external

    # firewall-cmd --get-zone-of-interface=enp1s0
    internal

Por defecto, la zona **internal** tiene habilitados estos servicios...

    # firewall-cmd --zone=internal --list-services
    ssh mdns samba-client dhcpv6-client

Vamos a agregarle los servicios de DNS, DHCP y Squid...

    # firewall-cmd --zone=internal --add-service=dns --add-service=dhcp --add-service=squid --add-service=cockpit

Verifique...

    # firewall-cmd --zone=internal --list-services
    ssh mdns samba-client dhcpv6-client dns dhcp squid cockpit

Luego, liste los servicos en **external**...

    # firewall-cmd --zone=external --list-services
    ssh

Retire el servicio OpenSSH de **external**...

    # firewall-cmd --zone=external --remove-service=ssh

El *enmascaramiento* debe estar en el dispositivo a la WAN...

    # firewall-cmd --zone=external --add-masquerade
    # firewall-cmd --zone=internal --remove-masquerade

Revise toda la configuración con los comandos...

    # firewall-cmd --zone=internal --list-all
    # firewall-cmd --zone=external --list-all

Ahora sí. Haga estos cambios permanentes...

    # firewall-cmd --runtime-to-permanent

### Continuación...

[Vaya a la parte 5, proxy Squid]({filename}/apuntes/fedora-server-28-parte-05/fedora-server-28-parte-05.md).
