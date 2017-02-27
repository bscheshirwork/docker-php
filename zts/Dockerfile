FROM php:zts
# -t bscheshir/php:zts .
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
