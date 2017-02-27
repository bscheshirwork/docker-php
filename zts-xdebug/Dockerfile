FROM php:zts
# -t bscheshir/php:zts-xdebug .
MAINTAINER BSCheshir <bscheshir.work@gmail.com>

RUN apt-get update && apt-get install -y \
        coreutils \
        re2c \
        autoconf \
        git \
        libpq-dev \
        --no-install-recommends \
  && rm -r /var/lib/apt/lists/* \
  && docker-php-ext-install -j$(nproc) pdo_pgsql \
  && apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false \
  && git clone https://github.com/krakjoe/pthreads.git \
    && ( \
        cd pthreads \
        && phpize \
        && ./configure --enable-pthreads \
        && make -j$(nproc) \
        && make install \
    ) \
    && rm -r pthreads \
    && docker-php-ext-enable pthreads \
  && curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

ENV XDEBUG_VERSION 2.5.1

RUN curl -fsSL "http://xdebug.org/files/xdebug-$XDEBUG_VERSION.tgz" -o xdebug-$XDEBUG_VERSION.tar.gz \
    && mkdir -p xdebug-$XDEBUG_VERSION \
    && tar -xf xdebug-$XDEBUG_VERSION.tar.gz -C xdebug-$XDEBUG_VERSION --strip-components=1 \
    && rm xdebug-$XDEBUG_VERSION.tar.gz \
    && ( \
        cd xdebug-$XDEBUG_VERSION \
        && phpize \
        && ./configure --enable-xdebug \
        && make -j$(nproc) \
        && make install \
    ) \
    && rm -r xdebug-$XDEBUG_VERSION \
    && docker-php-ext-enable xdebug

RUN echo -e "xdebug.remote_port = 9001\n\
xdebug.idekey = \"PHPSTORM\"\n\
xdebug.remote_enable=on" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
