Title: Instalación de Docker CE en Fedora
Slug: fedora-docker-ce
Summary: Docker Community Edition es la que tiene soporte libre por la comunidad y que nos brinda una versión más reciente que la de los repositorios de Fedora.
Tags: fedora, docker
Date: 2019-02-22 11:45
Modified: 2019-02-22 11:45
Category: apuntes
Preview: fedora-docker-ce-preview.jpg
Image: fedora-docker-ce-image.jpg


<img class="img-fluid" src="fedora-docker-ce-banner.jpg" alt="Fedora Docker CE">

### ¿Qué es Docker CE?

CE es por _Community Edition_. Es la versión *SIN PAGA* de Docker con soporte de la [comunidad](https://www.docker.com/docker-community). El número de verisón de Docker CE es más nuevo que la dada por los repositorios de Fedora.

En cambio, [Docker Enterprise](https://www.docker.com/products/docker-enterprise) es el mismo software con soporte y asistencia técnica profesional, bajo el modelo de paga por suscripción.

### Si tiene instalado la versión proporcionada por los repositorios de Fedora

Detener antes de desinstalar

    # systemctl stop docker

Desinstalar el Docker de Fedora

    # dnf remove docker-compose
    # dnf remove docker \
        docker-client \
        docker-client-latest \
        docker-common \
        docker-latest \
        docker-latest-logrotate \
        docker-logrotate \
        docker-selinux \
        docker-engine-selinux \
        docker-engine

### Para instalar Docker CE

Agregar el repositorio de Docker CE

    # dnf config-manager \
        --add-repo \
        https://download.docker.com/linux/fedora/docker-ce.repo

Revisar que esté habilitado

    # dnf repolist
    docker-ce-stable

Instalar

    # dnf install docker-ce docker-ce-cli containerd.io

Arrancar

    # systemctl start docker

Revise la versión

    $ docker version
    $ docker run hello-world
