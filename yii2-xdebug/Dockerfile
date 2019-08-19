FROM php:fpm
# docker build -t bscheshir/php:fpm-4yii2-xdebug --pull -- .
MAINTAINER BSCheshir <bscheshir.work@gmail.com>

# libfreetype6-dev libjpeg-dev libmcrypt-dev libpng-dev - for gd
# zlib1g-dev libicu-dev - for intl
# libpq-dev - for pdo_pgsql
# libmagickwand-dev - for imagick
# zlib1g-dev libmemcached-dev - for memcached

RUN apt-get update && \
    buildDependencies=" \
        git \
        zlib1g-dev \
        libmemcached-dev \
    " && \
    doNotUninstall=" \
        libmemcached11 \
        libmemcachedutil2 \
    " && \
    apt-get install -y $buildDependencies --no-install-recommends && \
    rm -r /var/lib/apt/lists/* && \
    \
    docker-php-source extract && \
    git clone https://github.com/php-memcached-dev/php-memcached /usr/src/php/ext/memcached/ && \
    docker-php-ext-install memcached && \
    \
    docker-php-source delete && \
    apt-mark manual $doNotUninstall && \
    apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false $buildDependencies

RUN apt-get update && \
    mainDependencies=" \
        git \
        zip \
        libzip-dev \
        zlib1g-dev \
        libfreetype6-dev \
        libjpeg-dev \
        libmcrypt-dev \
        libpng-dev \
        libicu-dev \
        libpq-dev \
        libmagickwand-dev \
    " && \
    apt-get install -y $mainDependencies \
    --no-install-recommends && \
    docker-php-ext-configure intl && \
    docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ --with-png-dir=/usr/include/ && \
    docker-php-ext-configure pgsql --with-pgsql=/usr/local/pgsql && \
    docker-php-ext-install -j$(nproc) pdo_mysql pdo_pgsql pgsql intl gd zip && \
# imagick
    pecl install imagick && \
    docker-php-ext-enable imagick && \
# apcu
    docker-php-source extract && \
    pecl install apcu && \
    docker-php-ext-enable apcu && \
    docker-php-source delete && \
    rm -r /tmp/pear/* && \
    echo "expose_php = off\n\
apc.enable_cli = 1\n\
opcache.optimization_level=0x7FFFBBFF\n\
cgi.fix_pathinfo = 0" >> /usr/local/etc/php/php.ini

ENV VERSION_PRESTISSIMO_PLUGIN=^0.3.0 \
    COMPOSER_ALLOW_SUPERUSER=1
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer \
    && composer global require --optimize-autoloader \
        "hirak/prestissimo:${VERSION_PRESTISSIMO_PLUGIN}" \
    && composer global dumpautoload --optimize

ENV XDEBUG_VERSION 2.8.0beta1
RUN pecl install xdebug-$XDEBUG_VERSION && \
    docker-php-ext-enable xdebug && \
    rm -r /tmp/pear/* && \
    echo -e "xdebug.remote_port = 9001\n\
xdebug.idekey = \"PHPSTORM\"\n\
xdebug.remote_enable=on" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
