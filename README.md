# sharedcar-docker

## Introducción

Este repositorio configura 4 contenedores para montar un entorno adecuado para un nuevo proyecto de Symfony 5

Los contenedores son los siguientes:

- `nginx:latest`
- `php:7.4-fpm`
- `mariadb:latest`
- `phpmyadmin/phpmyadmin:latest`

**nginx** y **php** son creados a partir de un Dockerfile que les incluye lo necesario para Symfony. Después los 4 son puestos en marcha por el `docker-composer.yml`

El directorio de trabajo en el contenedor (WORKDIR) es `/var/www/symfony`

Todo el proyecto se encuentra mapeado en nuestro directorio local `./files` (proyecto symfony) y `./database` (archivos de datos)

## Descarga

git clone https://github.com/davidbermudez/sharedcar-docker.git tu-proyecto

## Orquestando los contenedores

### Crea el siguiente archivo:

.env

        APP_NAME=your-project
        MYSQL_ROOT_PASSWORD=YourRootPass
        MYSQL_USER=compartecoche
        MYSQL_PASSWORD=YourUserPass
        MYSQL_DATABASE=compartecoche_db
        
### Crea los siguientes directorios

        `files/`
        `database/`
        
Verifica que tienes la siguiente estructura de archivos: 

        .
        ├── Dockerfile-nginx
        ├── Dockerfile-php
        ├── build
        │   └── nginx
        │       └── default.conf
        ├── database
        ├── files
        └── docker-compose.yml
        
`files` está fuera de este repositorio porque es el directorio mapeado con el directorio de trabajo del contenedor. Dentro de `files` tendrás otro repositorio independiente que contendrá tu proyecto en Symfony.

### Construir las imágenes y lanzar los contenedores:

Vamos a crear las imágenes definidas en Dockerfile-ngnix y Dockerfile-php:

        docker-compose build

Por último, lanzamos los contenedores:

        docker-compose up -d

## Crear el proyecto de symfony:

Si todo ha ido bien tendremos a los cuatro contenedores funcionando:

        docker ps --format "table {{.ID}}\t{{.Status}}\t{{.Ports}}\t{{.Names}}"
        CONTAINER ID   STATUS          PORTS                                                                        NAMES
        437aad26b21f   Up 31 minutes   0.0.0.0:8000->80/tcp, :::8000->80/tcp, 0.0.0.0:8443->443/tcp, :::8443->443/tcp    your-project_nginx
        cfc39947f8d0   Up 31 minutes   0.0.0.0:8080->80/tcp                                                              your-project_phpmyadmin
        b68c91b63ac8   Up 31 minutes   9000/tcp                                                                          your-project_php
        90c266d4ff62   Up 31 minutes   3306/tcp                                                                          your-project_db

Accedemos al contenedor `your-project_php`

        docker exec -it your-project_php bash
        root@3fa9b676624c:/var/www/symfony#
        
Configurar datos del usuario de git (el comando `symfony new` crea un nuevo repositorio)

        git config --global user.email "you@example.com"
        git config --global user.name "Your Name"
        git config --global credential.helper 'cache --timeout 3600'

Iniciar un nuevo proyecto de Symfony 5

        symfony new tu-proyecto --full
        
(Automáticamente te generará un nuevo repositorio de git dentro del contenedor)
