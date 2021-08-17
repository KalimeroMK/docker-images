FROM php:fpm-alpine3.14

RUN apk add icu-dev musl-dev zip libzip-dev freetype-dev libpng libpng-dev libjpeg-turbo-dev libxml2 libxslt-dev libwebp-dev zlib-dev libxpm-dev --no-cache \
    coreutils \
    php7-fpm \
    php7-apcu \
    php7-ctype \
    php7-curl \
    php7-dom \
    php7-gd \
    php7-iconv \
    php7-imagick \
    openssh-client \
    imagemagick \
    imagemagick-libs \
    imagemagick-dev \
    php7-json \
    php7-intl \
    php7-mcrypt \
    php7-fileinfo\
    php7-mbstring \
    php7-opcache \
    php7-openssl \
    php7-pdo \
    php7-pdo_mysql \
    php7-mysqli \
    php7-xml \
    php7-zlib \
    php7-phar \
    php7-tokenizer \
    php7-session \
    php7-simplexml \
    php7-xdebug \
    php7-zip \
    php7-xmlwriter \
    make \
    curl
RUN apk add --no-cache $PHPIZE_DEPS && pecl install xdebug && docker-php-ext-enable xdebug
RUN apk add --update nodejs npm
RUN apk add --update npm
# Install PHP extensions
RUN docker-php-ext-configure intl
RUN docker-php-ext-install intl
RUN docker-php-ext-configure gd --with-jpeg --with-webp --with-freetype \
    && docker-php-ext-install gd
RUN docker-php-ext-install mysqli pdo pdo_mysql
RUN docker-php-ext-configure zip
RUN docker-php-ext-install zip
RUN docker-php-ext-install pcntl
RUN docker-php-ext-install soap

# Get latest Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# Set working directory
WORKDIR /var/www