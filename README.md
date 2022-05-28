# sharedcar-docker

## Orquestando los contenedores

### Crea los siguientes archivos:

1. .env

        APP_NAME=sharedcar
        MYSQL_ROOT_PASSWORD=YourRootPass
        MYSQL_USER=compartecoche
        MYSQL_PASSWORD=YourUserPass
        MYSQL_DATABASE=compartecoche_db

2. Dockerfile-nginx

        FROM nginx:latest

        COPY ./build/nginx/default.conf /etc/nginx/conf.d/

        RUN apt update
        RUN apt install -y --no-install-recommends \
                python3-acme \
                python3-certbot \
                python3-mock  \
                python3-openssl \
                python3-pkg-resources \
                python3-pyparsing \
                python3-zope.interface

        RUN apt install -y --no-install-recommends python3-certbot-nginx

3. Dockerfile-php

        FROM php:7.4-fpm

        RUN apt-get update && apt-get install -y
        RUN apt-get install -y --no-install-recommends \
                git \
                zlib1g-dev \
                libxml2-dev \
                libzip-dev \
                libpq-dev \
                nano \
            && docker-php-ext-install \
                zip \
                intl \
                pdo \
                mysqli \
                pdo_mysql \
                opcache
        # Install Composer !
        RUN curl -sS https://getcomposer.org/installer | php && mv composer.phar /usr/local/bin/composer
        RUN curl -sS https://get.symfony.com/cli/installer | bash
        RUN mv /root/.symfony/bin/symfony /usr/local/bin/symfony

        # Set the default directory inside the container
        WORKDIR /var/www/symfony
4. docker-compose.yml

        version: '3.7'

        services:
          db:
            image: mariadb:latest
            container_name: ${APP_NAME}_db
            environment: # env
              MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
              MYSQL_DATABASE: ${MYSQL_DATABASE}
              MYSQL_USER: ${MYSQL_USER}
              MYSQL_PASSWORD: ${MYSQL_PASSWORD}
            volumes:
              - sql:/var/lib/mysql/
            networks:
              - app_net

          phpmyadmin:
            image: phpmyadmin/phpmyadmin:latest
            container_name: ${APP_NAME}_phpmyadmin
            ports:
              - 8080:80
            environment:
              PMA_HOST: db
              MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
            networks:
              - app_net

          php:
            container_name: ${APP_NAME}_php
            depends_on:
              - db
            build:
              context: .
              dockerfile: Dockerfile-php
            environment:
              APP_ENV: dev
              APP_DEBUG: 1
            volumes:
              - files:/var/www/symfony
            networks:
              - app_net

          nginx:
            container_name: ${APP_NAME}_nginx
            depends_on:
              - php
            build:
              context: .
              dockerfile: Dockerfile-nginx
            ports:
              - 8000:80 # So you'll access to Nginx with localhost:8000
              - 8443:443 # Same for HTTPS
              volumes:
              - files:/var/www/symfony
            networks:
              - ${APP_NAME}_net

        networks:
          app_net:

        volumes:
          files:
            driver: local
            driver_opts:
              type: 'none'
              o: 'bind'
              device: home/ubuntu/symfony
          sql:
            driver: local
            driver_opts:
              type: 'none'
              o: 'bind'
              device: home/ubuntu/symfony/database

5. build/nginx/default.conf

        server {
            listen 80;

            #listen 443 ssl;
            #listen [::]:443 ssl;
            #ssl_certificate /etc/ssl/certs/localhost.crt;
            #ssl_certificate_key /etc/ssl/private/localhost.key;

            root /var/www/symfony/public;
            server_name localhost;

            index index.php index.html;

            location / {
                try_files $uri /index.php$is_args$args;
            }

            location ~ \.php$ {
                fastcgi_pass php:9000; # Same name as the PHP service (php)
                fastcgi_index index.php;
                fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                include fastcgi_params;
            }

            error_log /var/log/nginx/myapp.error.log;
            access_log /var/log/nginx/myapp.access.log;
        }

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
        b68c91b63ac8   Up 31 minutes   9000/tcp                                                                          sharedcar_php
        90c266d4ff62   Up 31 minutes   3306/tcp                                                                          sharedcar_db

Accedemos al contenedor php

        docker exec -it sharedcar_php bash
        root@3fa9b676624c:/var/www/symfony#
        

Configurar datos del usuario de git (el comando symfony new crea un nuevo repositorio)

        git config --global user.email "you@example.com"
        git config --global user.name "Your Name"
        git config --global credential.helper 'cache --timeout 3600'

Creamos la aplicación

        root@3fa9b676624c:/var/www/symfony# symfony new sharedcar
        * Creating a new Symfony project with Composer
          (running /usr/local/bin/composer create-project symfony/skeleton /var/www/symfony/sharedcar  --no-interaction)

        * Setting up the project under Git version control
          (running git init /var/www/symfony/sharedcar)


         [OK] Your project is now ready in /var/www/symfony/sharedcar

        root@3fa9b676624c:/var/www/symfony#
