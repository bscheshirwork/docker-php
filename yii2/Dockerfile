FROM php:fpm
# docker build -t bscheshir/php:fpm-4yii2 .
MAINTAINER BSCheshir <bscheshir.work@gmail.com>

# libfreetype6-dev libjpeg62-turbo-dev libmcrypt-dev libpng12-dev - for gd
# zlib1g-dev libicu-dev - for intl
# libpq-dev - for pdo_pgsql
# libmagickwand-dev - for imagick
# libz-dev libmemcached-dev - for memcached

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
    git clone --branch php7 https://github.com/php-memcached-dev/php-memcached /usr/src/php/ext/memcached/ && \
    docker-php-ext-install memcached && \
    \
    docker-php-source delete && \
    apt-mark manual $doNotUninstall && \
    apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false $buildDependencies

RUN apt-get update && \
    mainDependencies=" \
        git \
        zip \
        zlib1g-dev \
        libfreetype6-dev \
        libjpeg62-turbo-dev \
        libmcrypt-dev \
        libpng12-dev \
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
cgi.fix_pathinfo = 0" >> /usr/local/etc/php/php.ini

ENV VERSION_PRESTISSIMO_PLUGIN=^0.3.0 \
    COMPOSER_ALLOW_SUPERUSER=1
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer \
    && composer global require --optimize-autoloader \
        "hirak/prestissimo:${VERSION_PRESTISSIMO_PLUGIN}" \
    && composer global dumpautoload --optimize
