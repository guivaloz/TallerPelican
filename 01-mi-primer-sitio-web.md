
# 01 mi primer sitio web

Verificar dónde está el ejecutable de Python y listar las librerías

    $ which python
    $ pip list

Crear un entorno virtual

    $ virtualenv -p python3 Pelican

Cambiar al directorio del Pelican

    $ cd Pelican

Activar el entorno

    $ . bin/activate

Escribir un archivo de requerimientos

    $ nano requirements.txt

Con este contenido

    pelican==4.0.1
    Markdown==3.1

Instalar requerimientos

    $ pip install -r requirements.txt

Verificar de nuevo

Ejecutar pelican-quickstart

    $ pelican-quickstart

Alimentar asistente

    > Where do you want to create your new web site? [.] MiPrimerSitioWeb
    > What will be the title of this web site? Mi primer sitio web
    > Who will be the author of this web site? Nombre Apellido
    > What will be the default language of this web site? [es]
    > Do you want to specify a URL prefix? e.g., https://example.com   (Y/n) n
    > Do you want to enable article pagination? (Y/n) n
    > What is your time zone? [Europe/Paris] America/Mexico_City
    > Do you want to generate a tasks.py/Makefile to automate generation and publishing? (Y/n) Y
    > Do you want to upload your website using FTP? (y/N) n
    > Do you want to upload your website using SSH? (y/N) n
    > Do you want to upload your website using Dropbox? (y/N) n
    > Do you want to upload your website using S3? (y/N) n
    > Do you want to upload your website using Rackspace Cloud Files? (y/N) n
    > Do you want to upload your website using GitHub Pages? (y/N) n
    Done. Your new project is available at...

Cambiar al directorio del sitio

    $ cd MiPrimerSitioWeb

Cambiar al directorio para el contenido

    $ cd content

En esa ubicación cree un archivo primer-pagina.md

    Title: Mi primer artículo
    Date: 2019-03-31 19:10
    Category: Vida cotidiana

    Esta es la primer página que escribo en el constructor de sitios web Pelican.

Baje un directorio

    $ cd ..

Consultar la ayuda

    $ make help

    Usage:
    make html                           (re)generate the web site
    make clean                          remove the generated files
    make regenerate                     regenerate files upon modification
    make publish                        generate using production settings
    make serve [PORT=8000]              serve site at http://localhost:8000
    make serve-global [SERVER=0.0.0.0]  serve (as root) to :80
    make devserver [PORT=8000]          serve and regenerate together
    make ssh_upload                     upload the web site via SSH
    make rsync_upload                   upload the web site via rsync+ssh

Crear sitio web

    $ make html

Levantar el servidor

    $ make serve

Ingrese en su navegador en

    http://127.0.0.1:8000

Termine el servidor con CTRL-C
