# Creating a development environment Symfony with Docker

## Requirements

**Check the Docker installation, with command :**

`docker -v`

**Result :**

`Docker version 20.10.21, build baeda1f82a`

<br>

**Check the Docker-compose installation, with command :**

`docker-compose -v`

**Result :**

`Docker Compose version 2.14.1`

---
## Creating of environment

### 1.Create the project directory

*Exemple*

```
mkdir myProjectEnv
cd myProjectEnv
```
### 2.Create and configure the docker-compose.yml

**Create new file : "docker-compose.yml"**

Add the images

*Copy the code for default environment*
```
version: '3'

services:
    db:
    image: mysql
    container_name: db_myblog_env
    restart: always
    volumes:
        - db-data:/var/lib/mysql
    environment:
        MYSQL_ALLOW_EMPTY_PASSWORD: 'yes'
    networks:
        - dev

phpmyadmin:
    image: phpmyadmin
    container_name: phpmyadmin_myblog_env
    restart: always
    depends_on:
        - db
    ports:
        - "8080:80"
    environment:
        PMA_HOST: db
    networks:
        - dev

www:
    build: docker
    container_name: www_symblog_youtube
    restart: always
    ports:
        - "8000:80"
    volumes:
        - ./docker/vhosts:/etc/apache2/sites-enabled
        - ./:/var/www
    networks:
        - dev


networks:
    dev:

volumes:
    db-data:
```

### 3.Creating the Dockerfile

**Create new folder : "Docker" and create new file "Dockerfile" in folder**

*Copy the code for default environment*

```
FROM php:8.1-apache

RUN echo "ServerName localhost" >> /etc/apache2/apache2.conf \
  \
  &&  apt-get update \
  &&  apt-get install -y --no-install-recommends \
  locales apt-utils git libicu-dev g++ libpng-dev libxml2-dev libzip-dev libonig-dev libxslt-dev unzip \
  \
  &&  echo "en_US.UTF-8 UTF-8" > /etc/locale.gen  \
  &&  echo "fr_FR.UTF-8 UTF-8" >> /etc/locale.gen \
  &&  locale-gen \
  \
  &&  curl -sS https://getcomposer.org/installer | php -- \
  &&  mv composer.phar /usr/local/bin/composer \
  \
  && curl -sL https://deb.nodesource.com/setup_18.x | bash \
  && apt-get install nodejs \
  \
  &&  curl -sS https://get.symfony.com/cli/installer | bash \
  &&  mv /root/.symfony/bin/symfony /usr/local/bin \
  \
  &&  docker-php-ext-configure \
  intl \
  &&  docker-php-ext-install \
  pdo pdo_mysql opcache intl zip calendar dom mbstring gd xsl \
  \
  &&  pecl install apcu && docker-php-ext-enable apcu

WORKDIR /var/www/
```

### 4.Creating the virtual host

**Create new folder : "vhosts" and create new file "vhosts.conf" in folder**

*Architecure project*
>myProjectEnv
><br>----Docker
><br>-------- vhosts
><br>------------ vhosts.conf
><br>--------- Dockerfile
><br>---- docker-compose.yml

*Copy the code for default environment*
```
<VirtualHost *:80>
ServerName localhost

    DocumentRoot /var/www/project/public
    DirectoryIndex /index.php

    <Directory /var/www/project/public>
        AllowOverride None
        Order Allow,Deny
        Allow from All

        FallbackResource /index.php
    </Directory>

    # uncomment the following lines if you install assets as symlinks
    # or run into problems when compiling LESS/Sass/CoffeeScript assets
    # <Directory /var/www/project>
    #     Options FollowSymlinks
    # </Directory>

    # optionally disable the fallback resource for the asset directories
    # which will allow Apache to return a 404 error when files are
    # not found instead of passing the request to Symfony
    <Directory /var/www/project/public/bundles>
        FallbackResource disabled
    </Directory>
    ErrorLog /var/log/apache2/project_error.log
    CustomLog /var/log/apache2/project_access.log combined

    # optionally set the value of the environment variables used in the application
    #SetEnv APP_ENV prod
    #SetEnv APP_SECRET <app-secret-id>
    #SetEnv DATABASE_URL "mysql://db_user:db_pass@host:3306/db_name"
</VirtualHost>
```

### 5.Test the environment 

**Execute command for testing the environment**

`docker-compose up -d`

>The first loading is a bit long for the creation of the containers

*Result*
```
[+] Running 4/4
⠿ Network myblogenv_dev            Created                                      0.0s
⠿ Container db_myblog_env          Started                                      0.5s
⠿ Container www_myblog_env         Started                                      0.4s
⠿ Container phpmyadmin_myblog_env  Started  
```

**Go to test local**

>Check URL
> <br> localhost:8080 or 127.0.0.1:8080 for PHPMyAdmin
> <br> localhost:8000 or 127.0.0.1:8000 for the Apache server 

## Congrate your Docker development environment for Symfony is complete !