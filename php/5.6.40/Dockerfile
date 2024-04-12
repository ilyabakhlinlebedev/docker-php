FROM debian:bookworm

ARG DEBIAN_FRONTEND="noninteractive"

USER root:root

WORKDIR /root/

RUN apt-get update \
    && apt-get -y upgrade \
    && apt-get -y dist-upgrade \
    && apt-get clean \
    && apt-get -y autoclean \
    && apt-get -y autoremove
