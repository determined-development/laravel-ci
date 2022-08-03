FROM ubuntu:22.04

LABEL maintainer="Determined Development"

ARG WWWGROUP=${WWWGROUP:-1000}
ARG WWWUSER=${WWWUSER:-1000}
ARG NODE_VERSION=${NODE_VERSION:-16}

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
    && apt-get install -y php8.1-cli php8.1-dev \
       php8.1-sqlite3 php8.1-gd \
       php8.1-curl \
       php8.1-imap php8.1-mysql php8.1-mbstring \
       php8.1-xml php8.1-zip php8.1-bcmath php8.1-soap \
       php8.1-intl php8.1-readline \
       php8.1-ldap \
       php8.1-msgpack php8.1-igbinary \
       php8.1-memcached php8.1-pcov php8.1-xdebug \
    && php -r "readfile('https://getcomposer.org/installer');" | php -- --install-dir=/usr/bin/ --filename=composer \
    && curl -sLS https://deb.nodesource.com/setup_$NODE_VERSION.x | bash - \
    && apt-get update \
    && apt-get install -y nodejs \
    && npm install -g npm \
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

COPY start-container /usr/local/bin/start-container
COPY build-env /usr/local/bin/build-env
RUN chmod +x /usr/local/bin/start-container
RUN chmod +x /usr/local/bin/build-env

ENTRYPOINT ["start-container"]