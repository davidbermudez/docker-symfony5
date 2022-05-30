# sharedcar-docker

## Descarga

git clone https://github.com/davidbermudez/sharedcar-docker.git tu-proyecto

## Orquestando los contenedores

### Crea el siguiente archivo:

.env

        APP_NAME=sharedcar
        MYSQL_ROOT_PASSWORD=YourRootPass
        MYSQL_USER=compartecoche
        MYSQL_PASSWORD=YourUserPass
        MYSQL_DATABASE=compartecoche_db

Verifica que tienes la siguiente estructura de archivos. En caso contrario crea los directorios correspondientes:

        .
        ├── Dockerfile-nginx
        ├── Dockerfile-php
        ├── build
        │   └── nginx
        │       └── default.conf
        ├── database
        ├── files
        └── docker-compose.yml

### Construir las imágenes y lanzar los contenedores:

Vamos a crear las imágenes definidas en Dockerfile-ngnix y Dockerfile-php:

        docker-compose build

Por último, lanzamos los contenedores:

        docker-compose up -d

## Crear el proyecto de symfony:

Si todo ha ido bien tendremos tres contenedores funcionando:

        docker ps --format "table {{.ID}}\t{{.Status}}\t{{.Ports}}\t{{.Names}}"
        CONTAINER ID   STATUS          PORTS                                                                        NAMES
        437aad26b21f   Up 31 minutes   0.0.0.0:8000->80/tcp, :::8000->80/tcp, 0.0.0.0:8443->443/tcp, :::8443->443/tcp    sharedcar_nginx
        cfc39947f8d0   Up 31 minutes   0.0.0.0:8080->80/tcp                                                              sharedcar_phpmyadmin
        b68c91b63ac8   Up 31 minutes   9000/tcp                                                                          sharedcar_php
        90c266d4ff62   Up 31 minutes   3306/tcp                                                                          sharedcar_db

Accedemos al contenedor php

        docker exec -it sharedcar_php bash
        root@3fa9b676624c:/var/www/symfony#
        

Configurar datos del usuario de git (el comando symfony new crea un nuevo repositorio)

        git config --global user.email "you@example.com"
        git config --global user.name "Your Name"
        git config --global credential.helper 'cache --timeout 3600'

Descargamos la aplicación

        git clone https://github.com/davidbermudez/sharedcar.git 
        
Instalamos vendors

        composer install
