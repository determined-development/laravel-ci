FROM ubuntu:22.04

LABEL maintainer="Determined Development"

ARG WWWGROUP=${WWWGROUP:-1000}
ARG WWWUSER=${WWWUSER:-1000}
ARG NODE_VERSION=${NODE_VERSION:-20}
ARG PHP_VERSION=${PHP_VERSION:-"8.3"}
ARG POSTGRES_VERSION=${POSTGRES_VERSION:-15}

WORKDIR /var/www/html

ENV TZ=UTC
ENV DEBIAN_FRONTEND=noninteractive

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

COPY start-container /usr/local/bin/start-container
COPY build-env /usr/local/bin/build-env
COPY composer-path /etc/profile.d/composer-path
RUN chmod +x /usr/local/bin/start-container \
    && chmod a+x /usr/local/bin/build-env \
    && chmod a+x /etc/profile.d/composer-path

RUN apt-get update \
    && apt-get install -y gnupg gosu curl ca-certificates zip unzip git supervisor sqlite3 libcap2-bin libpng-dev python2 mysql-client \
    &&  mkdir -p ~/.gnupg \
    && chmod 600 ~/.gnupg \
    && echo "disable-ipv6" >> ~/.gnupg/dirmngr.conf \
    && echo "keyserver hkp://keyserver.ubuntu.com:80" >> ~/.gnupg/dirmngr.conf \
    && gpg --recv-key 0x14aa40ec0831756756d7f66c4f4ea0aae5267a6c \
    && gpg --export 0x14aa40ec0831756756d7f66c4f4ea0aae5267a6c > /usr/share/keyrings/ppa_ondrej_php.gpg \
    && echo "deb [signed-by=/usr/share/keyrings/ppa_ondrej_php.gpg] https://ppa.launchpadcontent.net/ondrej/php/ubuntu jammy main" > /etc/apt/sources.list.d/ppa_ondrej_php.list \
    && curl -sS https://www.postgresql.org/media/keys/ACCC4CF8.asc | gpg --dearmor | tee /etc/apt/keyrings/pgdg.gpg >/dev/null \
    && echo "deb [signed-by=/etc/apt/keyrings/pgdg.gpg] http://apt.postgresql.org/pub/repos/apt jammy-pgdg main" > /etc/apt/sources.list.d/pgdg.list \
    && apt-get -y autoremove \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

RUN groupadd --force -g $WWWGROUP laravelci
RUN useradd -ms /bin/bash --no-user-group -g $WWWGROUP -u $WWWUSER laravelci
RUN cat /etc/profile.d/composer-path >> /home/laravelci/.profile

ENV CACHE_DRIVER=${CACHE_DRIVER:-"array"}
ENV SESSION_DRIVER=${SESSION_DRIVER:-"file"}
ENV QUEUE_CONNECTION=${QUEUE_CONNECTION:-"sync"}
ENV MAIL_MAILER=${MAIL_MAILER:-"array"}
ENV SRC_PATH=${SRC_PATH:-"/var/www/html"}

ONBUILD RUN curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg \
    && echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_$NODE_VERSION.x nodistro main" > /etc/apt/sources.list.d/nodesource.list \
    && apt-get update \
    && apt-get install -y "php$PHP_VERSION-cli" "php$PHP_VERSION-dev" \
       "php$PHP_VERSION-sqlite3" "php$PHP_VERSION-gd" \
       "php$PHP_VERSION-curl" \
       "php$PHP_VERSION-imap" "php$PHP_VERSION-mysql" "php$PHP_VERSION-mbstring" \
       "php$PHP_VERSION-xml" "php$PHP_VERSION-zip" "php$PHP_VERSION-bcmath" "php$PHP_VERSION-soap" \
       "php$PHP_VERSION-intl" "php$PHP_VERSION-readline" \
       "php$PHP_VERSION-ldap" \
       "php$PHP_VERSION-msgpack" "php$PHP_VERSION-igbinary" \
       "php$PHP_VERSION-memcached" "php$PHP_VERSION-pcov" "php$PHP_VERSION-xdebug" \
    && apt-get install -y nodejs \
    && apt-get install -y postgresql-client-$POSTGRES_VERSION \
    && php -r "readfile('https://getcomposer.org/installer');" | php -- --install-dir=/usr/bin/ --filename=composer \
    && apt-get -y autoremove \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

ENTRYPOINT ["start-container"]