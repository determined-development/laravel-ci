FROM ubuntu:22.04

LABEL maintainer="Determined Development"

ARG WWWGROUP=${WWWGROUP:-1000}
ARG WWWUSER=${WWWUSER:-1000}
ARG NODE_VERSION=${NODE_VERSION:-16}
ARG MYSQL_SOCKET_DIRECTORY='/var/run/mysqld'

WORKDIR /var/www/html

ENV TZ=UTC
ENV DEBIAN_FRONTEND=noninteractive

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

RUN apt-get update \
    && apt-get install -y gnupg gosu curl ca-certificates zip unzip git supervisor sqlite3 libcap2-bin libpng-dev python2 \
    && mkdir -p ~/.gnupg \
    && chmod 600 ~/.gnupg \
    && echo "disable-ipv6" >> ~/.gnupg/dirmngr.conf \
    && echo "keyserver hkp://keyserver.ubuntu.com:80" >> ~/.gnupg/dirmngr.conf \
    && gpg --recv-key 0x14aa40ec0831756756d7f66c4f4ea0aae5267a6c \
    && gpg --export 0x14aa40ec0831756756d7f66c4f4ea0aae5267a6c > /usr/share/keyrings/ppa_ondrej_php.gpg \
    && echo "deb [signed-by=/usr/share/keyrings/ppa_ondrej_php.gpg] https://ppa.launchpadcontent.net/ondrej/php/ubuntu jammy main" > /etc/apt/sources.list.d/ppa_ondrej_php.list \
    && apt-get update \
    && apt-get install -y php7.4-cli php7.4-dev \
       php7.4-sqlite3 php7.4-gd \
       php7.4-curl \
       php7.4-imap php7.4-mysql php7.4-mbstring \
       php7.4-xml php7.4-zip php7.4-bcmath php7.4-soap \
       php7.4-intl php7.4-readline \
       php7.4-ldap \
       php7.4-msgpack php7.4-igbinary \
       php7.4-memcached php7.4-pcov php7.4-xdebug \
    && php -r "readfile('https://getcomposer.org/installer');" | php -- --install-dir=/usr/bin/ --filename=composer \
    && curl -sLS https://deb.nodesource.com/setup_$NODE_VERSION.x | bash - \
    && apt-get update \
    && apt-get install -y nodejs \
    && npm install -g npm \
    && echo mysql-server mysql-server/root-pass password 'password' | debconf-set-selections \
    && echo mysql-server mysql-server/re-root-pass password 'password' | debconf-set-selections \
    && apt-get install -y mysql-server mysql-client \
    && mkdir -p $MYSQL_SOCKET_DIRECTORY \
    && chown root:mysql $MYSQL_SOCKET_DIRECTORY \
    && chmod 774 $MYSQL_SOCKET_DIRECTORY \
    && apt-get -y autoremove \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

ENV DB_DATABASE=${DB_DATABASE:-"testing"}
ENV DB_USERNAME=${DB_USERNAME:-"laravel"}
ENV DB_PASSWORD=${DB_PASSWORD:-"password"}
ENV CACHE_DRIVER=${CACHE_DRIVER:-"array"}
ENV SESSION_DRIVER=${SESSION_DRIVER:-"file"}
ENV QUEUE_CONNECTION=${QUEUE_CONNECTION:-"file"}
ENV MAIL_MAILER=${MAIL_MAILER:-"array"}

RUN groupadd --force -g $WWWGROUP laravelci
RUN useradd -ms /bin/bash --no-user-group -g $WWWGROUP -u 1337 laravelci

COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf
COPY start-container /usr/local/bin/start-container
RUN chmod +x /usr/local/bin/start-container

ENTRYPOINT ["start-container"]