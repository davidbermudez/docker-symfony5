# docker-symfony5
Deploy symfony 5 in minutes with Docker

## Introducción

Este proyecto pone en marcha 4 contenedores con todo lo necesario para crear un nuevo proyecto de Symfony 5. 

Los contenedores son los siguientes:

- `nginx:latest`
- `php:7.4-fpm`
- `mariadb:latest`
- `phpmyadmin/phpmyadmin:latest`

**nginx** y **php** son creados a partir de las imágenes definidas en los Dockerfile's `Dockerfile-ngnix` y `Dockerfile-php` que les incluye lo necesario para Symfony. Después el fichero `docker-composer.yml` crea la red y los volúmenes necesarios para que todo funcione.

## Pasos previos

Lo primero es **crear un directorio** en la máquina local o remota desde donde se van a correr la aplicación. Por ejemplo: `/home/user/symfony-project`.

A continuación nos situamos en dicho directorio y **descargamos** (hacemos clone) desde github este repositorio

        git clone https://github.com/davidbermudez/sharedcar-docker.git my-project
        
### Crea el archivo y los directorios siguientes:

Crea en el raiz de tu proyecto un archivo de configuración con el nombre `.env`

`/home/user/symfony-project/my-project/.env`

        APP_NAME=my-project
        MYSQL_ROOT_PASSWORD=YourRootPass
        MYSQL_USER=my-project-user
        MYSQL_PASSWORD=YourUserPass
        MYSQL_DATABASE=my-project_db
        PATH_TO_PROJECT=full-path-to-project-in-local # /home/user/symfony-project/my-project/
        
### Crea los siguientes directorios

        files/
        database/
        
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

El proyecto de Symfony se creará y ejecutará en el contenedor de php. El directorio de trabajo en dicho contenedor (WORKDIR) es `/var/www/symfony`

Todo el proyecto se encuentra mapeado en nuestro directorio local `./files` (proyecto symfony) y `./database` (archivos de datos)

## Orquestando los contenedores

### Configura nginx

En el archivo de configuración de nginx `build/nginx/default.conf` recuerda cambiar el root por el nombre de tu proyecto

        ...
        root /var/www/symfony/my-project/public;
        ...

### Construir las imágenes y lanzar los contenedores:

Vamos a crear las imágenes definidas en Dockerfile-ngnix y Dockerfile-php (sólo tienes que hacerlo la primera vez):

        docker-compose build

Por último, lanzamos los contenedores:

        docker-compose up -d

Docker-compose crea una red interna con IP fijas para los 4 contenedores:

        my-project_db          172.20.0.2        
        my-project_phpmyadmin  172.20.0.3
        my-project_php         172.20.0.4
        my-project_nginx       172.20.0.5
        
Importante la IP del contenedor de la base de datos, porque nos hará falta más adelante

## Preparando tu entorno de Symfony 5:

Si todo ha ido bien tendremos a los cuatro contenedores funcionando:

        docker ps --format "table {{.ID}}\t{{.Status}}\t{{.Ports}}\t{{.Names}}"
        CONTAINER ID   STATUS          PORTS                                           NAMES
        437aad26b21f   Up 31 minutes   0.0.0.0:8000->80/tcp, 0.0.0.0:8443->443/tcp,    my-project_nginx
        cfc39947f8d0   Up 31 minutes   0.0.0.0:8080->80/tcp                            my-project_phpmyadmin
        b68c91b63ac8   Up 31 minutes   9000/tcp                                        my-project_php
        90c266d4ff62   Up 31 minutes   3306/tcp                                        my-project_db

Accedemos al contenedor `my-project_php`

        docker exec -it my-project_php bash
        root@3fa9b676624c:/var/www/symfony#
        
Configurar datos del usuario de git (el comando `symfony new` crea un repositorio, con el código de nuestro proyecto de symfony)

        git config --global user.email "you@example.com"
        git config --global user.name "Your Name"
        git config --global credential.helper 'cache --timeout 3600'

## Iniciar un nuevo proyecto de Symfony 5 (o clonar uno existente)

### Iniciar un nuevo proyecto de Symfony 5

        symfony new tu-proyecto --full
        
(Automáticamente te generará un nuevo repositorio de git dentro del contenedor)

### Clonar

En el directorio /var/www/symfony de tu contenedor clonar el proyecto desde GitHub:

        git clone https://github.com/davidbermudez/the-fast-track.git
        
Accede al directorio

        cd the-fast-track

Instala las dependencias

        symfony composer install

## Configurar la base de datos

Verifica la IP de tu contenedor docker `my-project_db` y utiliza las variables declaradas en `.env` para editar el archivo `.env.local` de tu proyecto en symfony (Recuerda la IP del contenedor de base de datos:

        DATABASE_URL="mysql://the-fast-track:pass_user@172.20.0.2:3306/the-fast-track?serverVersion=mariadb.10.8.3&charset=utf8mb4"

### Recomendaciones:

- Crea una nueva rama y desarrolla en ella. Pasa sólo a producción (rama `main`) cuando compruebes que todo funciona

**¡Ya tienes tu proyecto Symfony funcionando!**
