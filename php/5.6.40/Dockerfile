FROM debian:bookworm AS base

ARG DEBIAN_FRONTEND="noninteractive"

USER root:root

WORKDIR /root/

COPY --chmod=644 --chown=root:root /etc/apt/sources.list.d/debian.sources /etc/apt/sources.list.d/debian.sources
COPY --chmod=644 --chown=root:root /etc/apt/sources.list.d/debian-security.sources /etc/apt/sources.list.d/debian-security.sources

RUN apt-get update \
    && apt-get -y upgrade \
    && apt-get -y dist-upgrade \
    && apt-get clean \
    && apt-get -y autoclean \
    && apt-get -y autoremove \
    && groupadd -g 1000 administrator \
    && useradd -c "Ilya Bakhlin Lebedev" -d /home/ilyabakhlinlebedev -g administrator -m -s /bin/bash -u 1000 ilyabakhlinlebedev

FROM base AS build

RUN apt-get install -y \
    build-essential \
    wget

FROM build AS build-ncurses

RUN wget -O - -q https://github.com/mirror/ncurses/archive/refs/tags/v6.4.tar.gz | tar -C /root/ -o -v -x -z

WORKDIR /root/ncurses-6.4/

RUN ./configure --prefix=/usr/local && make && make install

FROM build AS build-readline

RUN wget -O - -q https://git.savannah.gnu.org/cgit/readline.git/snapshot/readline-8.2.tar.gz | tar -C /root/ -o -v -x -z

WORKDIR /root/readline-8.2/

RUN ./configure --prefix=/usr/local && make && make install

FROM build AS build-php

COPY --from=build-ncurses /usr/local/ /usr/local/
COPY --from=build-readline /usr/local/ /usr/local/

RUN wget -O - -q https://www.php.net/distributions/php-5.6.40.tar.gz | tar -C /root/ -o -v -x -z

WORKDIR /root/php-5.6.40/

RUN ./configure \
    --disable-all \
    --disable-cgi \
    --prefix=/usr/local \
    --with-config-file-path=/usr/local/etc \
    --with-config-file-scan-dir=/usr/local/etc/ext \
    --with-readline

RUN make -j 4

FROM build AS install-php

COPY --from=build-php /usr/local/lib/libreadline.so.8.2 /usr/local/lib/libreadline.so.8.2

RUN ln -s -v /usr/local/lib/libreadline.so.8.2 /usr/local/lib/libreadline.so.8

RUN ln -s -v /usr/local/lib/libreadline.so.8.2 /usr/local/lib/libreadline.so

COPY --from=build-php /root/php-5.6.40/ /root/php-5.6.40/

WORKDIR /root/php-5.6.40/

RUN make install-cli

COPY --chmod=644 --chown=root:root /usr/local/etc/php.ini /usr/local/etc/php.ini

RUN mkdir -m 755 -v /usr/local/etc/ext

FROM base AS main

COPY --from=install-php /usr/local/ /usr/local/

USER ilyabakhlinlebedev:administrator

WORKDIR /home/ilyabakhlinlebedev/

ENTRYPOINT ["/usr/local/bin/php"]

CMD ["-a"]
