FROM debian:bookworm AS base

ARG DEBIAN_FRONTEND="noninteractive"

USER root:root

WORKDIR /root/

COPY --chmod=644 --chown=root:root /etc/apt/sources.list.d/debian.sources /etc/apt/sources.list.d/debian.sources
COPY --chmod=644 --chown=root:root /etc/apt/sources.list.d/debian-security.sources /etc/apt/sources.list.d/debian-security.sources

RUN apt-get update \
    && apt-get -y upgrade \
    && apt-get -y dist-upgrade \
    && apt-get --no-install-recommends -y install 7zip unzip \
    && apt-get clean \
    && apt-get -y autoclean \
    && apt-get -y autoremove \
    && groupadd -g 1000 administrator \
    && useradd -c "Ilya Bakhlin Lebedev" -d /home/ilyabakhlinlebedev -g administrator -m -s /bin/bash -u 1000 ilyabakhlinlebedev

FROM base AS build

RUN apt-get install -y \
    autoconf \
    build-essential \
    wget

FROM build AS build-libxml2

RUN wget -O - -q https://download.gnome.org/sources/libxml2/2.9/libxml2-2.9.4.tar.xz | tar -C /root/ -J -o -v -x

WORKDIR /root/libxml2-2.9.4/

RUN ./configure --prefix=/usr/local && make && make install

FROM build AS build-ncurses

RUN wget -O - -q https://github.com/mirror/ncurses/archive/refs/tags/v6.4.tar.gz | tar -C /root/ -o -v -x -z

WORKDIR /root/ncurses-6.4/

RUN ./configure --prefix=/usr/local && make && make install

FROM build AS build-openssl

RUN wget -O - -q https://www.openssl.org/source/old/1.0.2/openssl-1.0.2q.tar.gz | tar -C /root/ -o -v -x -z

WORKDIR /root/openssl-1.0.2q/

RUN ./config -fPIC --openssldir=/usr/local --prefix=/usr/local && make && make install

FROM build AS build-readline

RUN wget -O - -q https://git.savannah.gnu.org/cgit/readline.git/snapshot/readline-8.2.tar.gz | tar -C /root/ -o -v -x -z

WORKDIR /root/readline-8.2/

RUN ./configure --prefix=/usr/local && make && make install

FROM build AS build-zlib

RUN wget -O - -q https://github.com/madler/zlib/releases/download/v1.2.13/zlib-1.2.13.tar.gz | tar -C /root/ -o -v -x -z

WORKDIR /root/zlib-1.2.13/

RUN ./configure --prefix=/usr/local && make && make install

FROM build AS build-curl

COPY --from=build-openssl /usr/local/ /usr/local/
COPY --from=build-zlib /usr/local/ /usr/local/

RUN wget -O - -q https://curl.se/download/curl-7.52.1.tar.gz | tar -C /root/ -o -v -x -z

WORKDIR /root/curl-7.52.1/

RUN ./configure --prefix=/usr/local --with-ssl --with-zlib && make && make install

FROM build AS build-php

ARG LD_LIBRARY_PATH="/usr/local/lib"

COPY --from=build-curl /usr/local/ /usr/local/
COPY --from=build-libxml2 /usr/local/ /usr/local/
COPY --from=build-ncurses /usr/local/ /usr/local/
COPY --from=build-openssl /usr/local/ /usr/local/
COPY --from=build-readline /usr/local/ /usr/local/
COPY --from=build-zlib /usr/local/ /usr/local/

RUN wget -O - -q https://www.php.net/distributions/php-5.6.40.tar.gz | tar -C /root/ -o -v -x -z

WORKDIR /root/php-5.6.40/

RUN ./configure \
    --disable-all \
    --disable-cgi \
    --enable-dom \
    --enable-filter \
    --enable-hash \
    --enable-json \
    --enable-libxml \
    --enable-mbstring \
    --enable-phar \
    --enable-tokenizer \
    --enable-xml \
    --enable-xmlwriter \
    --enable-zip \
    --prefix=/usr/local \
    --with-config-file-path=/usr/local/etc \
    --with-config-file-scan-dir=/usr/local/etc/php \
    --with-curl \
    --with-readline \
    --with-libxml-dir=/usr/local \
    --with-openssl \
    --with-pear \
    --with-zlib

RUN make

FROM build AS install

COPY --from=build-php /usr/local/lib/libcurl.so.4.4.0 /usr/local/lib/libcurl.so.4.4.0
COPY --from=build-php /usr/local/lib/libreadline.so.8.2 /usr/local/lib/libreadline.so.8.2
COPY --from=build-php /usr/local/lib/libxml2.so.2.9.4 /usr/local/lib/libxml2.so.2.9.4

RUN ln -s -v /usr/local/lib/libcurl.so.4.4.0 /usr/local/lib/libcurl.so.4 \
    && ln -s -v /usr/local/lib/libreadline.so.8.2 /usr/local/lib/libreadline.so.8 \
    && ln -s -v /usr/local/lib/libxml2.so.2.9.4 /usr/local/lib/libxml2.so.2

RUN ln -s -v /usr/local/lib/libcurl.so.4.4.0 /usr/local/lib/libcurl.so \
    && ln -s -v /usr/local/lib/libreadline.so.8.2 /usr/local/lib/libreadline.so \
    && ln -s -v /usr/local/lib/libxml2.so.2.9.4 /usr/local/lib/libxml2.so

COPY --chmod=644 --chown=root:root /usr/local/etc/php.ini /usr/local/etc/php.ini

RUN wget -O /usr/local/share/ca-certificates/cacert.pem https://curl.se/ca/cacert.pem

COPY --from=build-php /root/php-5.6.40/ /root/php-5.6.40/

RUN mkdir -m 755 -v /usr/local/etc/php

FROM install AS install-php-extensions

WORKDIR /root/php-5.6.40/

RUN make install && pecl channel-update pecl.php.net

FROM install-php-extensions AS install-php-extensions-xdebug

RUN pecl install xdebug-2.5.5

FROM install AS install-php

WORKDIR /root/php-5.6.40/

RUN make install-cli

RUN wget -O - -q https://getcomposer.org/installer | php -- --filename=composer --install-dir=/usr/local/bin/

COPY --from=install-php-extensions-xdebug /usr/local/lib/php/extensions/no-debug-non-zts-20131226/xdebug.so /usr/local/include/xdebug.so

FROM base AS main

COPY --from=install-php /usr/local/bin/ /usr/local/bin/
COPY --from=install-php /usr/local/etc/ /usr/local/etc/
COPY --from=install-php /usr/local/include/ /usr/local/include/
COPY --from=install-php /usr/local/lib/ /usr/local/lib/

COPY --chmod=644 --chown=root:root /usr/local/etc/php/xdebug.ini /usr/local/etc/php/xdebug.ini

USER ilyabakhlinlebedev:administrator

WORKDIR /home/ilyabakhlinlebedev/

ENTRYPOINT ["/usr/local/bin/php"]

CMD ["-a"]
