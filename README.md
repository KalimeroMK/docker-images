Table of Contents
-------------------

 * [Installation](#installation)
 * [Quick Start](#quick-start)
 * [Persistence](#developmentpersistence)
 * [Linked to other container](#linked-to-other-container)
 * [Adding PHP-extension](#adding-php-extension) 
 * [Logging](#logging)
 * [Out of the box](#out-of-the-box)

Installation
-------------------

 * [Install Docker 1.9+](https://docs.docker.com/installation/) or [askubuntu](http://askubuntu.com/a/473720)
 * Pull the latest version of the image.
 
```bash
docker pull kalimeromk/apache:8.0
```

or other versions (7.4,7.3, 7.2, 7.1, 7.0, 5.6, 5.5, 5.4 or 5.3):


Alternately you can build the image yourself.

```bash
git clone https://github.com/KalimeroMK/docker-images
cd docker-apache-php
docker build -t="$USER/docker-apache-php" .
```

Quick Start
-------------------

Run the application container:

```bash
docker run --name app -d -p 8080:80 kalimeromk/apache:8.0
```

The simplest way to login to the app container is to use the `docker exec` command to attach a new process to the running container.

```bash
docker exec -it app bash
```

Development/Persistence
-------------------

For development a volume should be mounted at `/var/www/html/`.

The updated run command looks like this.

```bash
docker run --name app -d -p 8080:80 \
  -v /host/to/path/app:/var/www/html/ \
  kalimeromk/apache:8.0
```

This will make the development.

Linked to other container
-------------------

As an example, will link with RDBMS PostgreSQL. 

```bash
docker network create mariadb

docker run --name db -d mariadb:10.3
```

Run the application container:

```bash
docker run --name app -d -p 8080:80 \
  --net pg_net \
  -v /host/to/path/app:/var/www/www/ \
  kalimeromk/apache:8.0
```

Adding PHP-extension
-------------------

You can use one of two choices to install the required php-extensions:

1. `docker exec -it app bash -c 'apt-get update && apt-get install php-mongo && rm -rf /var/lib/apt/lists/*'`

2. Create your container on based the current. Сontents Dockerfile:

```
FROM ubuntu:bionic

RUN apt-get update \
    && apt-get install -y php-mongo \
    && rm -rf /var/lib/apt/lists/* 

WORKDIR /var/www/app/

EXPOSE 80 443

CMD ["/sbin/entrypoint.sh"]
```

Next step,

```bash
docker build -t php-5.6 .
docker run --name app -d -p 8080:80 php-5.6
```

>See installed php-extension: `docker exec -it app php -m`

>PHP-extension "Mcrypt" was REMOVED in PHP 7.2. Use [Sodium](http://php.net/manual/en/book.sodium.php) or [OpenSSL](http://php.net/manual/en/book.openssl.php)

Logging
-------------------

All the logs are forwarded to stdout and sterr. You have use the command `docker logs`.

```bash
docker logs app
```

####Split the logs

You can then simply split the stdout & stderr of the container by piping the separate streams and send them to files:

```bash
docker logs app > stdout.log 2>stderr.log
cat stdout.log
cat stderr.log
```

or split stdout and error to host stdout:

```bash
docker logs app > -
docker logs app 2> -
```

####Rotate logs

Create the file /etc/logrotate.d/docker-containers with the following text inside:

```
/var/lib/docker/containers/*/*.log {
    rotate 31
    daily
    nocompress
    missingok
    notifempty
    copytruncate
}
```
> Optionally, you can replace `nocompress` to `compress` and change the number of days.

Out of the box
-------------------
 * Ubuntu 12.04, 14.04, 16.04 or 18.04 LTS
 * Apache 2.4.x/2.2.x
 * PHP 5.3, 5.4, 5.5, 5.6, 7.0, 7.1, 7.2 or 7.3
 * Composer (package manager)

>Environment depends on the version of PHP.

>Example docker-compose.yml for apache
> 
```
version: '3'
networks:
  laravel:
services:
  mysql:
    platform: "linux/x86_64" // remove this line if not useing M1 chip
    image: mariadb:10.3
    container_name: mysql
    restart: unless-stopped
    tty: true
    ports:
      - "3306:3306"
    environment:
      MYSQL_DATABASE: homestead
      MYSQL_USER: homestead
      MYSQL_PASSWORD: secret
      MYSQL_ROOT_PASSWORD: secret
      SERVICE_TAGS: dev
      SERVICE_NAME: mysql
    networks:
      - laravel
  php:
    container_name: php
    image: kalimeromk/apache:7.4
    restart: unless-stopped
    ports:
      - 8080:80
    hostname: ogledalo.local
    volumes:
      - ./:/var/www/html
    networks:
      - laravel
  app:
    image: phpmyadmin/phpmyadmin
    container_name: phpmyadmin
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: secret
      PMA_HOST: mysql
      PMA_PORT: 3306
    links:
      - mysql
    ports:
      - 8085:80
    volumes:
      - /sessions
    networks:
      - laravel
```
>Example docker-compose.yml for nginx

```
version: '3'
services:

  #PHP Service
  app:
    image: kalimeromk/php:8.0
    container_name: app
    restart: unless-stopped
    tty: true
    environment:
      SERVICE_NAME: app
      SERVICE_TAGS: dev
    working_dir: /var/www
    volumes:
      - ./:/var/www
      - ./php/local.ini:/usr/local/etc/php/conf.d/local.ini
    networks:
      - app-network

  #Nginx Service
  webserver:
    image: nginx:alpine
    container_name: webserver
    restart: unless-stopped
    tty: true
    ports:
      - "8000:80"
      - "443:443"
    volumes:
      - ./:/var/www
      - ./nginx/conf.d/:/etc/nginx/conf.d/
    networks:
      - app-network

  #MySQL Service
  mysql:
    image: arm64v8/mariadb
    container_name: mysql
    restart: unless-stopped
    tty: true
    ports:
      - "3306:3306"
    environment:
      MYSQL_DATABASE: homestead
      MYSQL_USER: homestead
      MYSQL_PASSWORD: secret
      MYSQL_ROOT_PASSWORD: secret
      SERVICE_TAGS: dev
      SERVICE_NAME: mysql
    volumes:
      - dbdata:/var/lib/mysql/
    networks:
      - app-network

#Docker Networks
networks:
  app-network:
    driver: bridge
#Volumes
volumes:
  dbdata:
    driver: local
````
>default.conf file for nginx

```
server {
    listen 80;
    index index.php index.html;
    error_log  /var/log/nginx/error.log;
    access_log /var/log/nginx/access.log;
    root /var/www/public;
    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass app:9000;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }
    location / {
        try_files $uri $uri/ /index.php?$query_string;
        gzip_static on;
    }
}
```
-------------------

Apache + PHP docker image is open-sourced software licensed under the [MIT license](http://opensource.org/licenses/MIT)
