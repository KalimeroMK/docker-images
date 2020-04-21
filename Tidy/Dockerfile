FROM ubuntu:bionic
MAINTAINER KalimeroMK <zbogoevski@gmail.com>

ENV OS_LOCALE="en_US.UTF-8"
RUN apt-get update && apt-get install -y locales && locale-gen ${OS_LOCALE}
ENV LANG=${OS_LOCALE} \
    LANGUAGE=${OS_LOCALE} \
    LC_ALL=${OS_LOCALE} \
    DEBIAN_FRONTEND=noninteractive
ENV APACHE_CONF_DIR=/etc/apache2 \
    PHP_CONF_DIR=/etc/php/7.3 \
    PHP_DATA_DIR=/var/lib/php
ENV APACHE_DOCUMENT_ROOT /var/www/html/public
COPY . /var/www/html
COPY entrypoint.sh /sbin/entrypoint.sh
RUN \
    BUILD_DEPS='software-properties-common' \
    && dpkg-reconfigure locales \
    && apt-get install --no-install-recommends -y $BUILD_DEPS \
    && add-apt-repository -y ppa:ondrej/php \
    && add-apt-repository -y ppa:ondrej/apache2 \
    && add-apt-repository ppa:ondrej/php \
    && apt-get update \
    && apt-get install -y curl zip apache2 nano php-imagick libapache2-mod-php7.4 php7.4-cli php7.4-readline php7.4-mbstring php7.4-zip php7.4-intl php7.4-xml php7.4-json php7.4-curl php7.4-gd php7.4-pgsql php7.4-mysql php7.4-bcmath php7.4-ctype php7.4-pdo php7.4-json php7.4-tokenizer php-pear \
    # Apache settings
    && cp /dev/null ${APACHE_CONF_DIR}/conf-available/other-vhosts-access-log.conf \
    && rm ${APACHE_CONF_DIR}/sites-enabled/000-default.conf ${APACHE_CONF_DIR}/sites-available/000-default.conf \
    && a2enmod rewrite php7.4 \
    # Install composer
    && curl -sS https://getcomposer.org/installer | php -- --version=1.8.4 --install-dir=/usr/local/bin --filename=composer \
    # Cleaning
    && apt-get purge -y --auto-remove $BUILD_DEPS \
    && apt-get autoremove -y \
    && rm -rf /var/lib/apt/lists/* \
    # Forward request and error logs to docker log collector
    && ln -sf /dev/stdout /var/log/apache2/access.log \
    && ln -sf /dev/stderr /var/log/apache2/error.log \
    && chmod 755 /sbin/entrypoint.sh \
    && chown www-data:www-data ${PHP_DATA_DIR} -Rf
COPY ./configs/apache2.conf ${APACHE_CONF_DIR}/apache2.conf
COPY ./configs/app.conf ${APACHE_CONF_DIR}/sites-available/app.conf
RUN ln -s ${APACHE_CONF_DIR}/sites-available/app.conf ${APACHE_CONF_DIR}/sites-enabled/app.conf
COPY ./configs/php.ini  ${PHP_CONF_DIR}/apache2/conf.d/custom.ini
WORKDIR /var/www/html/
EXPOSE 80
# By default, simply start apache.
CMD ["/sbin/entrypoint.sh"]
