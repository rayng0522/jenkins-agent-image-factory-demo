FROM debian:stretch

USER root

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        apt-transport-https \
        ca-certificates \
        gss-ntlmssp \
        curl \
        gnupg2 \
        git \
        jq \
        gcc \
        python3-dev \
        python3-pip \
        python3-setuptools \
        python3-wheel \
        openssh-client \
        software-properties-common \
        unzip \
        less \
        locales \
        sudo \
    && rm -rf /var/lib/apt/lists/* \
    && pip3 --no-cache-dir install \
        azure-cli \
        yamllint \
    && useradd -ms /bin/bash builder \
    && echo "builder ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/builder

USER builder
WORKDIR /home/builder
