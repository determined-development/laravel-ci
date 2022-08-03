FROM php:8.0-cli

LABEL maintainer="Determined Development"

ARG WWWGROUP=${WWWGROUP:-1000}
ARG WWWUSER=${WWWUSER:-$UID}
ARG NODE_VERSION=${NODE_VERSION:-16}

WORKDIR /var/www/html

ENV TZ=UTC
ENV DEBIAN_FRONTEND=noninteractive

ENV DB_DATABASE=${DB_DATABASE:-"testing"}
ENV DB_USERNAME=${DB_USERNAME:-"laravel"}
ENV DB_PASSWORD=${DB_PASSWORD:-"password"}
ENV DB_HOST=${DB_HOST:-"localhost"}
ENV DB_PORT=${DB_HOST:-"3306"}
ENV CACHE_DRIVER=${CACHE_DRIVER:-"array"}
ENV SESSION_DRIVER=${SESSION_DRIVER:-"file"}
ENV QUEUE_CONNECTION=${QUEUE_CONNECTION:-"file"}
ENV MAIL_MAILER=${MAIL_MAILER:-"array"}

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

RUN apt-get update \
    && apt-get install -y gosu gnupg wget lsb-release \
    && wget https://dev.mysql.com/get/mysql-apt-config_0.8.23-1_all.deb \
    && dpkg -i mysql-apt-config_0.8.23-1_all.deb \
    && echo mysql-community-server mysql-community-server/root-pass password 'password' | debconf-set-selections \
    && echo mysql-community-server mysql-community-server/re-root-pass password 'password' | debconf-set-selections \
    && apt-get update \
    && apt-get install -y default-mysql-server default-mysql-client \
    && rm mysql-apt-config_0.8.23-1_all.deb

ADD https://github.com/mlocati/docker-php-extension-installer/releases/latest/download/install-php-extensions /usr/local/bin/

RUN chmod +x /usr/local/bin/install-php-extensions && \
    install-php-extensions \
        gd bcmath intl zip \
        igbinary msgpack \
        memcached xdebug pcov \
    && php -r "readfile('https://getcomposer.org/installer');" | php -- --install-dir=/usr/bin/ --filename=composer \
    && curl -sLS https://deb.nodesource.com/setup_$NODE_VERSION.x | bash - \
    && apt-get update \
    && apt-get install -y nodejs \
    && npm install -g npm \
    && apt-get -y autoremove \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

RUN groupadd --force -g $WWWGROUP laravelci
RUN useradd -ms /bin/bash --no-user-group -g $WWWGROUP -u 1337 laravelci

COPY start-container /usr/local/bin/start-container
COPY prepare-env /usr/local/bin/prepare-env

ENTRYPOINT ["start-container"]