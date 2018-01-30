FROM php:zts
# docker build -t bscheshir/php:zts-xdebug --pull -- .
MAINTAINER BSCheshir <bscheshir.work@gmail.com>

RUN apt-get update && apt-get install -y \
        coreutils \
        re2c \
        autoconf \
        git \
        libpq-dev \
        --no-install-recommends && \
    rm -r /var/lib/apt/lists/* && \
    docker-php-ext-configure pgsql --with-pgsql=/usr/local/pgsql && \
    docker-php-ext-install -j$(nproc) pdo_pgsql pgsql && \
    apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false && \
    git clone https://github.com/krakjoe/pthreads.git && \
    ( \
        cd pthreads && \
        phpize && \
        ./configure --enable-pthreads && \
        make -j$(nproc) && \
        make install \
    ) && \
    rm -r pthreads && \
    docker-php-ext-enable pthreads && \
    curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

ENV XDEBUG_VERSION 2.6.0
RUN pecl install xdebug-$XDEBUG_VERSION && \
    docker-php-ext-enable xdebug && \
    rm -r /tmp/pear/* && \
    echo -e "xdebug.remote_port = 9001\n\
xdebug.idekey = \"PHPSTORM\"\n\
xdebug.remote_enable=on" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
