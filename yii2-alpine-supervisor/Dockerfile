FROM php:fpm-alpine3.18
# docker build -t bscheshir/php:fpm-alpine-4yii2-supervisor --pull -- .
MAINTAINER BSCheshir <bscheshir.work@gmail.com>

# Install system packages & PHP extensions required for Yii 2.0 Framework
RUN apk --update add \
        git \
        curl \
        curl-dev \
        bash \
        bash-completion \
        oniguruma-dev \
        libzip-dev \
        freetype-dev \
        icu \
        icu-dev \
        libxml2-dev \
        libintl \
        libjpeg-turbo-dev \
        libpng-dev \
        mysql-client \
        nodejs \
        postgresql-dev && \
    docker-php-ext-configure gd \
        --with-freetype \
        --with-jpeg && \
    docker-php-ext-configure bcmath && \
    docker-php-ext-configure pgsql --with-pgsql=/usr/local/pgsql && \
    docker-php-ext-install \
        soap \
        zip \
        curl \
        bcmath \
        exif \
        gd \
        intl \
        mbstring \
        opcache \
        pdo_mysql \
        pdo_pgsql \
        pgsql && \
    apk del \
        icu-dev \
        gcc \
        g++ && \
    apk add --no-cache tzdata && \
    set -ex && \
# memcache
    apk add --no-cache --virtual .memcached-deps zlib-dev libmemcached-dev cyrus-sasl-dev && \
    docker-php-source extract && \
    git clone https://github.com/php-memcached-dev/php-memcached /usr/src/php/ext/memcached/ && \
    docker-php-ext-install memcached && \
#    docker-php-source delete && \
    apk add --no-cache --virtual .memcached-runtime-deps libmemcached-libs && \
    apk del .memcached-deps && \
# imagick
    printf "\n%s\n%s" "@edge http://dl-cdn.alpinelinux.org/alpine/edge/main" "@testing http://dl-cdn.alpinelinux.org/alpine/edge/testing" >> /etc/apk/repositories && \
    apk --update add imagemagick-dev@edge imagemagick@edge && \
    apk add --no-cache --virtual .phpize-deps $PHPIZE_DEPS libtool && \
    export CFLAGS="$PHP_CFLAGS" CPPFLAGS="$PHP_CPPFLAGS" LDFLAGS="$PHP_LDFLAGS" && \
    pecl install imagick-3.5.0 && \
    docker-php-ext-enable imagick && \
#    apk del .phpize-deps && \
# apcu
#    apk add --no-cache --virtual .phpize-deps $PHPIZE_DEPS && \
#    docker-php-source extract && \
    pecl install apcu && \
    docker-php-ext-enable apcu && \
    docker-php-source delete && \
    apk del .phpize-deps && \
    rm -r /tmp/pear/* && \
    echo -e "expose_php = off\n\
upload_max_filesize	= 5M\n\
apc.enable_cli = 1\n\
opcache.optimization_level=0x7FFFBBFF\n\
cgi.fix_pathinfo = 0" >> /usr/local/etc/php/php.ini

# Convert PSR-0/4 autoloading to classmap to get a faster autoloader.
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer && \
    composer global require --optimize-autoloader

# supervisor
RUN apk add --no-cache \
        supervisor&& \
    mkdir /etc/supervisord.d && \
    mkdir -p /var/log/supervisor && \
    echo -e "[unix_http_server]\n\
file = /tmp/supervisor.sock\n\
chmod = 0777\n\
chown= nobody:nogroup\n\
\n\
[supervisord]\n\
logfile=/var/log/supervisor/supervisord.log\n\
logfile_maxbytes = 50MB\n\
logfile_backups=10\n\
loglevel = info\n\
pidfile = /tmp/supervisord.pid\n\
nodaemon = true\n\
umask = 022\n\
identifier = supervisor\n\
\n\
[supervisorctl]\n\
serverurl = unix:///tmp/supervisor.sock\n\
\n\
[rpcinterface:supervisor]\n\
supervisor.rpcinterface_factory = supervisor.rpcinterface:make_main_rpcinterface\n\
\n\
[include]\n\
files = /etc/supervisor/conf.d/*.conf" >> /etc/supervisord.conf

ENTRYPOINT ["supervisord", "--nodaemon", "--configuration", "/etc/supervisord.conf"]
