# php

Based at :whale:[official images](https://hub.docker.com/_/php/) 
and expand it.

Supported tags and respective `Dockerfile` links
================================================


## for yii2 

- `7.2.4-fpm-4yii2`, `fpm-4yii2` ([yii2/Dockerfile](./yii2/Dockerfile))
- `7.2.4-fpm-4yii2-xdebug`, `fpm-4yii2-xdebug` ([yii2-xdebug/Dockerfile](./yii2-xdebug/Dockerfile))
>note: based on `fpm-alpine3.7`
- `7.2.4-fpm-alpine-4yii2`, `fpm-alpine-4yii2` ([yii2-alpine/Dockerfile](./yii2-alpine/Dockerfile))
- `7.2.4-fpm-alpine-4yii2-xdebug`, `fpm-alpine-4yii2-xdebug` ([yii2-alpine-xdebug/Dockerfile](./yii2-alpine-xdebug/Dockerfile))
- `7.2.4-fpm-alpine-4yii2-supervisor`, `fpm-alpine-4yii2-supervisor` ([yii2-alpine-supervisor/Dockerfile](./yii2-alpine-supervisor/Dockerfile))
- `7.2.4-fpm-alpine-4yii2-supervisor-xdebug`, `fpm-alpine-4yii2-supervisor-xdebug` ([yii2-alpine-supervisor-xdebug/Dockerfile](./yii2-alpine-supervisor-xdebug/Dockerfile))

FROM `php:fpm`

added `composer` (global require `hirak/prestissimo:^0.3.0`)

added `yii2 dependences` (all pass requirements.php, :information_source: ApcCache: Alternatively `APCu PHP` extension could be used via setting `useApcu` to `true` )

tag: `{sourceref}-4yii2`

added `xdebug 2.7.0alpha1`

tag: `{sourceref}-4yii2-xdebug`

`docker pull bscheshir/php:7.2.4-fpm-4yii2-xdebug`

## for zts 

- `7.2.4-zts`, `zts` ([zts/Dockerfile](./zts/Dockerfile))
- `7.2.4-zts-xdebug`, `zts-xdebug` ([zts-xdebug/Dockerfile](./zts-xdebug/Dockerfile))


FROM `php:zts`

added `pthreads`

tag: `{sourceref}-zts`

added xdebug

tag: `{sourceref}-zts-xdebug`


## Usage
### Example for yii2 [docker-compose.yml](https://github.com/bscheshirwork/docker-yii2-app-advanced/blob/master/docker-run/docker-compose.yml)
```
version: '2'
services:
  php:
    image: bscheshir/php:7.2.4-fpm-4yii2-xdebug
    restart: always
    volumes:
      - ../php-code:/var/www/html #php-code
    depends_on:
      - db
    environment:
      TZ: Europe/Moscow
      XDEBUG_CONFIG: "remote_host=dev-Aspire-V3-772 remote_port=9001 var_display_max_data=1024 var_display_max_depth=5"
      PHP_IDE_CONFIG: "serverName=yii2advanced"
  nginx:
    image: nginx:1.13.11-alpine
    restart: always
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
  mysql:
    image: mysql:8.0.4
    restart: always
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
    restart: always
    volumes:
      - ../mysql-proxy-conf:/opt/mysql-proxy/conf
      - ../mysql-proxy-logs:/opt/mysql-proxy/logs
    environment:
      PROXY_DB_HOST:
      PROXY_DB_PORT: 3306
      REMOTE_DB_HOST: mysql
      REMOTE_DB_PORT: 3306
      LUA_SCRIPT: "/opt/mysql-proxy/conf/log.lua"
      LOG_FILE: "/opt/mysql-proxy/logs/mysql.log"
    depends_on:
      - mysql
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
    image: bscheshir/php:7.2.4-fpm-alpine-4yii2-supervisor-xdebug
    restart: always
    volumes:
      - ../php-code:/var/www/html #php-code
      - ../supervisor-conf:/etc/supervisor/conf.d
      - ../supervisor-logs:/var/log/supervisor
    depends_on:
      - db
    environment:
      TZ: Europe/Moscow
      XDEBUG_CONFIG: "remote_host=dev-Aspire-V3-772 remote_port=9003 var_display_max_data=1024 var_display_max_depth=5"
      PHP_IDE_CONFIG: "serverName=yii2advanced"
```


### Example zts [docker-compose.yml](https://github.com/bscheshirwork/multispider/blob/master/zts/docker-compose.yml)
```
version: '2'
services:
  php:
    image: bscheshir/php:7.2.4-zts
    restart: always
    hostname: phphost
    working_dir: /multispider
    depends_on:
      - db
    volumes:
      - ..:/multispider #php-code
      - ~:/home/user
  db:
    image: postgres:10.3-alpine
    restart: always
    volumes:
      - ../.db:/var/lib/postgresql/data #DB-data
    environment:
      POSTGRES_PASSWORD: multispider
      POSTGRES_DB: multispider
      POSTGRES_USER: multispider
```

## How to build manually 

Clone or fetch
```sh
git clone https://github.com/bscheshirwork/docker-php.git
cd docker-php
git pull
```

Build and push. For example `bscheshir/php:fpm-alpine-4yii2-xdebug` - this image will be used in [docker-codeception-yii2](https://github.com/bscheshirwork/docker-codeception-yii2)
```
docker build -t bscheshir/php:7.2.4-fpm-alpine-4yii2-xdebug -t bscheshir/php:fpm-alpine-4yii2-xdebug --pull -- ./yii2-alpine-xdebug
docker push bscheshir/php:7.2.4-fpm-alpine-4yii2-xdebug
docker push bscheshir/php:fpm-alpine-4yii2-xdebug
```