Title: Instalación de Fedora Server 28, parte 5, proxy Squid
Slug: fedora-server-28-parte-05
Summary: Vamos a configurar un proxy con Squid.
Tags: fedora, gnu linux, servidores, software libre
Date: 2018-06-24 22:15
Modified: 2018-06-24 22:15
Category: apuntes
Preview: fedora-logo-icon.png


### Squid - archivos de configuración

Instale Squid...

    # dnf install squid

En el directorio `/etc/squid` se deben de encontrar los archivos de configuración.

    # cd /etc/squid

Ahí debe estar el archivo `squid.conf`. Éste puede cargar otros archivos *conf*, *txt*, *css*, etc.

Como puede ser muy extensa la configuración de Squid, vamos a dividir la configuración de `squid.conf` en archivos según su propósito. Comienze por el principal que cargará los demás...

    # nano squid.conf

Con este contenido...

    # Correo electronico
    cache_mgr webmaster@miredlocal.lan

    # Nombre de este servidor
    visible_hostname miproxy

    # Archivos separados de configuracion
    include "/etc/squid/squid-01-declaraciones.conf"
    include "/etc/squid/squid-02-accesos.conf"
    include "/etc/squid/squid-03-opciones.conf"
    include "/etc/squid/squid-04-balanceo.conf"

### Squid - declaraciones

Cree `squid-01-declaraciones.conf`...

    # nano squid-01-declaraciones.conf

Ajuste el contenido de acuerdo a sus necesidades...

    # Puerto para permitir https
    acl SSLPorts  port  443        # https
    acl SSLPorts  port 5222        # .whatsapp.com, .whatsapp.net
    acl SSLPorts  port 5228        # mtalk.google.com

    # Puertos que se pueden usar
    acl SafePorts port   21        # ftp
    acl SafePorts port   70        # gopher
    acl SafePorts port   80        # http
    acl SafePorts port  443        # https
    acl SafePorts port 1025-65535  # unregistered ports

    # Declaración connect
    acl CONNECT method CONNECT

    # Desde y hacia Localhost
    acl DesdeLocalhost    src        127.0.0.0/8
    acl HaciaLocalhost    dst        127.0.0.0/8

    # Desde y hacia LAN
    acl DesdeLAN          src        192.168.4.0/24
    acl HaciaLAN          dst        192.168.4.0/24

    # Definiciones de sitios
    acl SitiosPermitidos  dstdomain  "/etc/squid/sitios-permitidos.txt"
    acl SitiosProhibidos  dstdomain  "/etc/squid/sitios-prohibidos.txt"

    # Definiciones de expresiones regulares sobre publicidad
    acl RegExpPublicidad  dstdom_regex -i "/etc/squid/regexp-publicidad.txt"

    # Audio MP3
    acl AudioMP3Mime      rep_mime_type  audio/mpeg
    acl AudioMP3RegExp    urlpath_regex  \.mp3$

    # Video MPEG4 (MPG4)
    acl VideoMP4Mime      rep_mime_type  video/mp4 video/x-mp4
    acl VideoMP4RegExp    urlpath_regex  \.mp4$

    # Flash Video Format (FLV)
    acl FlashVideoMime    rep_mime_type  video/flv video/x-flv
    acl FlashVideoRegExp  urlpath_regex  \.flv$

    # Flash General Media Scripts (SWF)
    acl FlashMime         rep_mime_type  application/x-shockwave-flash
    acl FlashRegExp       urlpath_regex  \.swf$

### Squid - accesos

Cree `squid-02-accesos.conf`...

    # nano squid-02-accesos.conf

Ajuste el contenido de acuerdo a sus necesidades...

    # El administrador del cache 'manager' solo lo hará localhost
    http_access         allow  DesdeLocalhost manager
    http_access         deny                  manager

    # NEGAR solicitudes por los puertos NO seguros
    http_access         deny   !SafePorts

    # NEGAR CONNECT a otros puertos NO SSL seguros
    http_access         deny   CONNECT !SSLPorts

    # Es recomendable negar conexiones a 127.0.0.1, éstas deben hacerce fuera del proxy
    http_access         deny   HaciaLocalhost

    # NEGAR Adobe Flash
    http_access         deny   FlashRegExp

    # Permitir sitios permitidos
    http_access         allow  SitiosPermitidos

    # NEGAR prohibidos y publicidad
    http_access         deny   SitiosProhibidos
    http_access         deny   RegExpPublicidad

    # Permitir todo lo demás a localhost
    http_access         allow  DesdeLocalhost

    # Permitir equipos LAN
    http_access         allow  DesdeLAN

    # NEGAR TODO a todos los demás
    http_access         deny   all

### Squid - opciones

Cree `squid-03-opciones.conf`...

    # nano squid-03-opciones.conf

Ajuste el contenido de acuerdo a sus necesidades...

    # Puerto
    http_port 3128

    # Disk cache directory
    # - ufs es el sistema de almacenamiento propio de Squid
    # - /var/cache/squid es la ruta
    # - 8192 es la cantidad en MB, debe tener como maximo 20% menos del tamano de la particion
    #        la partición /var tiene 16 GB, así que 8 GB es la mitad
    # -   64 es L1, el numero de subdirectorios de primer nivel
    # -  256 es L2, el numero de subdirectorios de segundo nivel
    cache_dir ufs /var/cache/squid 8192 64 256

    # Tamano maximo por objeto
    maximum_object_size 128 MB

    # Cantidad de Memoria RAM que se usara antes de escribir en el disco
    cache_mem 256 MB
    cache_replacement_policy heap LFUDA

    # Patrones de archivos y sus tiempos de permanencia en el cache
    refresh_pattern ^ftp:             1440  20%  10080
    refresh_pattern ^gopher:          1440   0%   1440
    refresh_pattern -i (/cgi-bin/|\?)    0   0%      0
    refresh_pattern .                    0  20%   4320

    # Leave coredumps in the first cache dir
    coredump_dir /var/cache/squid

    # Tiempo de espera de cierre de conexiones
    # al disminuirlo el apagado sera mas breve
    shutdown_lifetime 12 seconds

### Squid - balanceo

Cree `squid-04-balanceo.conf`...

    # nano squid-04-balanceo.conf

Ajuste el contenido de acuerdo a sus necesidades...

    # Se usaran esta cantidad de cubos
    delay_pools 2

    # Hay tres tipos de clases
    # 1 - Tiene un unico cubo Global
    # 2 - Tiene un cubo Global y 256 cubos individuales
    # 3 - Tiene un cubo Global, 256 cubos de Red y 65536 individuales
    # Declaracion de las clases de los cubos
    #   delay_class N clase
    delay_class 1 1
    delay_class 2 2

    # Los tamanos de los cubos son en bytes
    # El contrato con Telmex es de hasta 10 Mb/s
    # 1250000 bytes/seg =  10.0 Mb/s
    #  625000 bytes/seg =   5.0 Mb/s
    #  312500 bytes/seg =   2.5 Mb/s
    #  250000 bytes/seg =   2.0 Mb/s
    #  125000 bytes/seg =   1.0 Mb/s

    # Cubo clase 1: 10/5 Mb/s
    # Pocos equipos y grandes descargas
    delay_parameters 1 1250000/625000

    # Cubo clase 2: 10/10 Mb/s y 2/2 Mb/s
    # Muchos equipos y carga balanceada
    delay_parameters 2 1250000/1250000 250000/250000

    # El primer cubo es para localhost
    delay_access 1 allow DesdeLocalhost

    # El segundo cubo es para la LAN
    delay_access 2 allow DesdeLAN

### Squid - stios permitidos y prohibidos

Baje las expresiones regulares para bloquear publicidad...

    # wget "http://pgl.yoyo.org/adservers/serverlist.php?hostformat=squid-dstdom-regex&showintro=0&mimetype=plaintext" -O regexp-publicidad.txt

Cree su lista de sitios de entera confianza `sitios-permitidos.txt`...

    .wikipedia.org
    .mediawiki.org
    wikimediafundation.org
    .ubuntu.com
    .fedoraproject.org
    fedoramagazine.org
    .github.com
    .osuosl.org
    .movimientolibre.com

Y su lista negra con sitios prohibidos `sitios-prohibidos.txt`...

    .microsoft.com
    .windowsupdate.com
    .msftncsi.com

### Políticas de SELinux

Prepare el directorio para el caché...

    # mkdir /var/cache/squid
    # chown squid /var/cache/squid
    # squid -z -N -f /etc/squid/squid.conf

Arranque el daemon...

    # systemctl start squid.service
    # systemctl status squid.service

En mi caso ha fallado porque las política de seguridad SELinux no permiten algunas acciones de Squid. Revise el estatus...

    # systemctl status squid

Me reporta este error...

    Ipc::Mem::Segment::create failed to shm_open

Al revisar los últimos registros en la bitácora...

    # tail /var/log/audit/audit.log

Aparece una negación...

    type=AVC msg=audit(1529629815.011:337): avc:  denied  { unlink } for  pid=2086 comm="squid" name="squid-cf__metadata.shm" dev="tmpfs" ino=32961 scontext=system_u:system_r:squid_t:s0 tcontext=unconfined_u:object_r:user_tmp_t:s0 tclass=file permissive=0

Si gusta profundizar en **SELinux** lea [Fedora 22 SELinux User's and Administrator's Guide](https://docs-old.fedoraproject.org/en-US/Fedora/22/html/SELinux_Users_and_Administrators_Guide/index.html).

**Opción 1:**: Crear una nueva política a partir de la bitácora. Instale `policycoreutils-python-utils`...

    # dnf install policycoreutils-python-utils

Como *root* nos vamos al directorio `/root` o donde podamos crear archivos...

    # cd /root

1) Ejecute `audit2allow` alimentado por la bitácora `audit.log`...

    # audit2allow -M MYSQUIDPOLICY < /var/log/audit/audit.log

2) Lea el texto de la política creada

    # cat MYSQUIDPOLICY.te

3) Cargue la política, **note que se usa el archivo pp**...

    # semodule -i MYSQUIDPOLICY.pp

4) Arranque el daemon...

    # systemctl start squid.service
    # systemctl status squid.service

5) Vuelva al paso uno si ha fallado el arranque de Squid.

Tuve que **repetir siete veces** este proceso para dar con la política correcta, que mostrando `MYSQUIDPOLICY7.te ` es...

**Opción 2:**: Cree la política a partir del archivo de texto.

Instale **policycoreutils-devel**...

    # dnf install policycoreutils-devel

Cree `DnsmasqSquidPolicy.te`...

    # cd /root
    # nano DnsmasqSquidPolicy.te

Con este contenido...

    module DnsmasqSquidPolicy 1.0;

    require {
            type squid_t;
            type dnsmasq_t;
            type user_tmp_t;
            type var_t;
            class capability dac_override;
            class file { append create getattr open read rename unlink write };
    }

    #============= dnsmasq_t ==============

    allow dnsmasq_t self:capability dac_override;

    #============= squid_t ==============

    allow squid_t user_tmp_t:file unlink;
    allow squid_t var_t:file unlink;
    allow squid_t var_t:file { append create getattr open read rename write };

Compile e instale...

    # checkmodule -M -m -o DnsmasqSquidPolicy.mod DnsmasqSquidPolicy.te
    # semodule_package -o DnsmasqSquidPolicy.pp -m DnsmasqSquidPolicy.mod
    # semodule -i DnsmasqSquidPolicy.pp

Arranque el daemon...

    # systemctl start squid.service
    # systemctl status squid.service

### Arranque al encender y muro de fuego

Habilite que inicie el *daemon* al encender...

    # systemctl enable squid

Verifique que el muro de fuego tenga abierto el servicio **squid** en la LAN...

    # firewall-cmd --zone=internal --list-services
    ssh mdns samba-client dhcpv6-client cockpit dns squid dhcp

### Hacer que el servidor YA NO SEA un ruteador

Al estar habilitado el enmascaramiento, cualquier equipo puede acceder a Internet sin pasar por el proxy. Consulte con...

    # firewall-cmd --zone=external --query-masquerade

Deshabilite el enmascaramiento con...

    # firewall-cmd --zone=external --remove-masquerade

Haga pruebas con su navegador de internet en un equipo en la red local. Si configura *sin proxy* no debería tener Internet. Luego configure el proxy y pruebe que funcione.

Si tiene éxito, haga los cambios permanantes...

    # firewall-cmd --runtime-to-permanent

### Continuación...

[Vaya a la parte 6, configuración del proxy en los clientes]({filename}/apuntes/fedora-server-28-parte-06/fedora-server-28-parte-06.md).
