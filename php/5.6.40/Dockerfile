FROM debian:bookworm

ARG DEBIAN_FRONTEND="noninteractive"

USER root:root

WORKDIR /root/

COPY --chmod=644 --chown=root:root /etc/apt/sources.list.d/debian.sources /etc/apt/sources.list.d/debian.sources

RUN apt-get update \
    && apt-get -y upgrade \
    && apt-get -y dist-upgrade \
    && apt-get clean \
    && apt-get -y autoclean \
    && apt-get -y autoremove
