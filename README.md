# php

Based at :whale:[official images](https://hub.docker.com/_/php/) 
and expand it.

Supported tags and respective `Dockerfile` links
================================================


## for yii2  

- `8.2.11-fpm-4yii2`, `fpm-4yii2` ([yii2/Dockerfile](./yii2/Dockerfile))
- `8.2.11-fpm-4yii2-xdebug`, `fpm-4yii2-xdebug` ([yii2-xdebug/Dockerfile](./yii2-xdebug/Dockerfile))  
- `8.2.11-fpm-alpine-4yii2`, `fpm-alpine-4yii2` ([yii2-alpine/Dockerfile](./yii2-alpine/Dockerfile))
- `8.2.11-fpm-alpine-4yii2-xdebug`, `fpm-alpine-4yii2-xdebug` ([yii2-alpine-xdebug/Dockerfile](./yii2-alpine-xdebug/Dockerfile))
- `8.2.11-fpm-alpine-4yii2-supervisor`, `fpm-alpine-4yii2-supervisor` ([yii2-alpine-supervisor/Dockerfile](./yii2-alpine-supervisor/Dockerfile))
- `8.2.11-fpm-alpine-4yii2-supervisor-xdebug`, `fpm-alpine-4yii2-supervisor-xdebug` ([yii2-alpine-supervisor-xdebug/Dockerfile](./yii2-alpine-supervisor-xdebug/Dockerfile))

FROM `php:fpm`

added `composer` (global --optimize-autoloader)

added `yii2 dependences` (all pass requirements.php, :information_source: ApcCache: Alternatively `APCu PHP` extension could be used via setting `useApcu` to `true` )

tag: `{sourceref}-4yii2`

added `Xdebug 3.2.2`

tag: `{sourceref}-4yii2-xdebug`

`docker pull bscheshir/php:8.2.11-fpm-alpine-4yii2-xdebug`


## Usage
### Example for yii2 [docker-compose.yml](https://github.com/bscheshirwork/docker-yii2-app-advanced/blob/master/docker-run/docker-compose.yml)
```yml
version: '2'
services:
  php:
    image: bscheshir/php:8.2.11-fpm-alpine-4yii2-xdebug
    restart: unless-stopped
    volumes:
      - ../php-code:/var/www/html #php-code
      - ~/.composer/cache:/root/.composer/cache
    depends_on:
      - db
    environment:
      TZ: Europe/Moscow
      XDEBUG_CONFIG: "client_host=${DEV_REMOTE_HOST} client_port=${DEV_REMOTE_PORT}"
      PHP_IDE_CONFIG: "serverName=${DEV_SERVER_NAME}"
    extra_hosts:
      - "host.docker.internal:host-gateway"
  nginx:
    image: nginx:1.17-alpine
    restart: unless-stopped
    ports:
      - "8080:8080"
      - "8081:8081"
    depends_on:
      - php
    volumes_from:
      - php
    volumes:
      - ../nginx-conf:/etc/nginx/conf.d #nginx-conf
      - ../nginx-logs:/var/log/nginx #nginx-logs
    environment:
      TZ: Europe/Moscow
  mysql:
    image: mysql:8.0.18
    entrypoint: ['/entrypoint.sh', '--default-authentication-plugin=mysql_native_password'] # https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_default_authentication_plugin
    restart: unless-stopped
    expose:
      - "3306" #for service mysql-proxy
    ports:
      - "3307:3306" #for external connection
    volumes:
      - ../mysql-data/db:/var/lib/mysql #mysql-data
    environment:
      TZ: Europe/Moscow
      MYSQL_ROOT_PASSWORD: yii2advanced
      MYSQL_DATABASE: yii2advanced
      MYSQL_USER: yii2advanced
      MYSQL_PASSWORD: yii2advanced
  db: #mysql-proxy
    image: bscheshir/mysql-proxy:0.8.5
    expose:
      - "3306" #for service php
    ports:
      - "3308:3306" #for external connection
    restart: unless-stopped
    volumes:
      - ../mysql-proxy-conf:/opt/mysql-proxy/conf
      - ../mysql-proxy-logs:/opt/mysql-proxy/logs
    depends_on:
      - mysql
    environment:
      TZ: Europe/Moscow
      PROXY_DB_HOST:
      PROXY_DB_PORT: 3306
      REMOTE_DB_HOST: mysql
      REMOTE_DB_PORT: 3306
      LUA_SCRIPT: "/opt/mysql-proxy/conf/log.lua"
      LOG_FILE: "/opt/mysql-proxy/logs/mysql.log"
```

### Example run command

phpinfo | xdebug

from `docker-run` dir
```
docker-compose run --rm php -i |grep xdebug
```
or 
```
docker-compose -f ~/projects/yii2project/docker-run/docker-compose.yml run --rm php -i |grep xdebug
```

crontab (full path needed)
```
*/10 * * * * /usr/local/bin/docker-compose -f /home/user/projects/yii2project/docker-compose.yml run --rm php ./yii command/action
```


### Example `supervisor` [docker-compose.yml](https://github.com/bscheshirwork/docker-yii2-app-advanced-redis/blob/master/docker-run/docker-compose.yml)
```
version: '2'
services:
  php-supervisor: # for workers
    image: bscheshir/php:8.2.11-fpm-alpine-4yii2-supervisor-xdebug
    restart: unless-stopped
    volumes:
      - ../php-code:/var/www/html #php-code
      - ../supervisor-conf:/etc/supervisor/conf.d
      - ../supervisor-logs:/var/log/supervisor
    depends_on:
      - db
    environment:
      TZ: Europe/Moscow
      XDEBUG_CONFIG: "remote_host=host.docker.internal remote_port=9003 var_display_max_data=1024 var_display_max_depth=5"
      PHP_IDE_CONFIG: "serverName=yii2advanced"
    extra_hosts:
      - "host.docker.internal:host-gateway"
```

## How to build manually 

### Clone or get fresh
```sh
git clone https://github.com/bscheshirwork/docker-php.git
cd docker-php
git pull
```

### Build and push. 

For example `bscheshir/php:fpm-alpine-4yii2-xdebug` - this image will be used in [docker-codeception-yii2](https://github.com/bscheshirwork/docker-codeception-yii2)
and another `alpine` images
```sh
docker build -t bscheshir/php:8.2.11-fpm-alpine-4yii2-xdebug -t bscheshir/php:fpm-alpine-4yii2-xdebug --pull -- ./yii2-alpine-xdebug
docker push bscheshir/php:8.2.11-fpm-alpine-4yii2-xdebug
docker push bscheshir/php:fpm-alpine-4yii2-xdebug

docker build -t bscheshir/php:8.2.11-fpm-alpine-4yii2 -t bscheshir/php:fpm-alpine-4yii2 --pull -- ./yii2-alpine
docker push bscheshir/php:8.2.11-fpm-alpine-4yii2
docker push bscheshir/php:fpm-alpine-4yii2

docker build -t bscheshir/php:8.2.11-fpm-alpine-4yii2-supervisor-xdebug -t bscheshir/php:fpm-alpine-4yii2-supervisor-xdebug --pull -- ./yii2-alpine-supervisor-xdebug
docker push bscheshir/php:8.2.11-fpm-alpine-4yii2-supervisor-xdebug
docker push bscheshir/php:fpm-alpine-4yii2-supervisor-xdebug

docker build -t bscheshir/php:8.2.11-fpm-alpine-4yii2-supervisor -t bscheshir/php:fpm-alpine-4yii2-supervisor --pull -- ./yii2-alpine-supervisor
docker push bscheshir/php:8.2.11-fpm-alpine-4yii2-supervisor
docker push bscheshir/php:fpm-alpine-4yii2-supervisor
```

# User and groups

## alpine
We can assign standard `alpine` uid of `www-data` to new `local user` for granted access to some places from `php`.  
For example if you have `dev` group in Ubuntu:  
```sh
sudo useradd -u 82 -g dev -MN alpine-www-data
``` 
