Title: Instalación de Fedora Server 28, parte 6, configuración del proxy en los clientes
Slug: fedora-server-28-parte-06
Summary: Haremos que sea automática la carga de la configuración del proxy para los equipos en la red local.
Tags: fedora, gnu linux, servidores, software libre
Date: 2018-06-24 22:16
Modified: 2018-06-24 22:16
Category: apuntes
Preview: fedora-logo-icon.png


### Apache HTTPD

Instale Apache HTTPD si no lo ha hecho...

    # dnf install httpd

Cree el archivo `/etc/httpd/conf.d/proxy-autoconfig.conf`...

    # nano /etc/httpd/conf.d/proxy-autoconfig.conf

Con este contenido...

    AddType application/x-ns-proxy-autoconfig .pac
    AddType application/x-ns-proxy-autoconfig .dat

Vaya al directorio...

    # cd /var/www/html/

Donde vamos a crear los archivos `proxy.pac` y `wpad.dat` que definen si usar o no el proxy...

* Directo - 127.0.0.1
* Directo - 0.0.0.0
* Directo - Direcciones IP de la red local 192.168.4.0/255.255.255.0
* Directo - Dominios sin puntos, como http://equipo1
* Directo - Dominios de la red local, como http://proxy.miredlocal.lan
* Directo - WhatsApp Web usa Socks
* Todo lo demás al PROXY

Cree `proxy.pac`...

    # nano proxy.pac

Con este contenido (cambie `miredlocal.lan` y `192.168.4.0` por su configuración)...

    function FindProxyForURL(url, host) {

        /* Normalize the URL for pattern matching */
        url = url.toLowerCase();
        host = host.toLowerCase();

        /* Don't proxy local hostnames */
        if (isPlainHostName(host)) {
            return "DIRECT";
        }

        /* Don't proxy local domains */
        if (dnsDomainIs(host, "*.miredlocal.lan")) {
            return "DIRECT";
        }
        if (isInNet(host, "192.168.4.0", "255.255.255.0")) {
            return "DIRECT";
        }

        /* Don't proxy non-routable addresses (RFC 3330) */
        if (isResolvable(host)) {
            var hostIP = dnsResolve(host);
            if (isInNet(hostIP, '0.0.0.0', '255.0.0.0') ||
                isInNet(hostIP, '10.0.0.0', '255.0.0.0') ||
                isInNet(hostIP, '127.0.0.0', '255.0.0.0') ||
                isInNet(hostIP, '169.254.0.0', '255.255.0.0') ||
                isInNet(hostIP, '172.16.0.0', '255.240.0.0') ||
                isInNet(hostIP, '192.0.2.0', '255.255.255.0') ||
                isInNet(hostIP, '192.88.99.0', '255.255.255.0') ||
                isInNet(hostIP, '192.168.0.0', '255.255.0.0') ||
                isInNet(hostIP, '198.18.0.0', '255.254.0.0') ||
                isInNet(hostIP, '224.0.0.0', '240.0.0.0') ||
                isInNet(hostIP, '240.0.0.0', '240.0.0.0')) {
                return 'DIRECT';
            }
            return "PROXY proxy.miredlocal.lan:3128";
        }

        /* Internet */
        if (url.substring(0,5) == "http:" ||
            url.substring(0,6) == "https:" ||
            url.substring(0,4) == "ftp:") {
            return "PROXY proxy.miredlocal.lan:3128";
        }

        /* Default*/
        return "DIRECT";

    }

Cópielo a `wpad.dat`...

    # cp proxy.pac wpad.dat

Como está escrito previamente, en `/etc/hosts` está declarado "wpad" con la dirección IP del servidor...

    192.168.4.1  proxy wpad

Debe activar la línea `dhcp-option=252,...` en `/etc/dnsmasq.conf`...

    # nano /etc/dnsmasq.conf

Con el URL al archivo `proxy.pac`...

    # URL del script de configuración automática de proxy en el navegador
    dhcp-option=252,http://proxy.miredlocal.lan/proxy.pac
    # En cambio, para Windows, cuando no se tiene, se envía un avance de línea
    #dhcp-option=252,"\n"

Reinicie DNSmasq...

    # systemctl restart dnsmasq

Arranque o recargue el *daemon* Apache HTTPD...

    # systemctl start httpd

Agregue el servicio HTTP (TCP 80) al muro de fuego en la zona *interna*...

    # firewall-cmd --zone=internal --add-service=http
    # firewall-cmd --runtime-to-permanent

Haga estas pruebas desde un equipo en su red local...

    $ dig @192.168.4.1 wpad.miredlocal.lan
    $ dig @192.168.4.1 proxy.miredlocal.lan
    $ wget http://proxy.miredlocal.lan/proxy.pac
    $ wget http://wpad.miredlocal.lan/wpad.dat

### Continuación...

[Vaya a la parte 7, servidor de archivos SAMBA]({filename}/apuntes/fedora-server-28-parte-07/fedora-server-28-parte-07.md).
