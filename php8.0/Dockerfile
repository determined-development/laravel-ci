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
    && apt-get install -y php8.0-cli php8.0-dev \
       php8.0-sqlite3 php8.0-gd \
       php8.0-curl \
       php8.0-imap php8.0-mysql php8.0-mbstring \
       php8.0-xml php8.0-zip php8.0-bcmath php8.0-soap \
       php8.0-intl php8.0-readline \
       php8.0-ldap \
       php8.0-msgpack php8.0-igbinary \
       php8.0-memcached php8.0-pcov php8.0-xdebug \
    && apt-get -y autoremove \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

RUN php -r "readfile('https://getcomposer.org/installer');" | php -- --install-dir=/usr/bin/ --filename=composer

ENV CACHE_DRIVER=${CACHE_DRIVER:-"array"}
ENV SESSION_DRIVER=${SESSION_DRIVER:-"file"}
ENV QUEUE_CONNECTION=${QUEUE_CONNECTION:-"file"}
ENV MAIL_MAILER=${MAIL_MAILER:-"array"}
ENV NODE_VERSION=${NODE_VERSION:-16}

RUN groupadd --force -g $WWWGROUP laravelci
RUN useradd -ms /bin/bash --no-user-group -g $WWWGROUP -u $WWWUSER laravelci

COPY start-container /usr/local/bin/start-container
COPY build-env /usr/local/bin/build-env
COPY setup-node /usr/local/bin/setup-node
RUN chmod +x /usr/local/bin/start-container
RUN chmod +x /usr/local/bin/build-env
RUN chmod +x /usr/local/bin/setup-node

ENTRYPOINT ["start-container"]