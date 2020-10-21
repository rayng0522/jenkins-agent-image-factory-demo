FROM docker-rtsre-local.pruregistry.intranet.asia:8443/jenkins-agent-base:latest

ARG http_proxy
ARG https_proxy
ARG no_proxy

USER root

ARG ANSIBLE_VERSIONS="2.7.5 2.10.0"
ARG ANSIBLE_LINT_VERSION="4.2.0"

RUN apt-get update \
    && apt-get dist-upgrade -y \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* \
    && pip3 --no-cache-dir install \
        cffi \
        ansible-lint==${ANSIBLE_LINT_VERSION} \
        pywinrm

RUN mkdir /ansible \
    && mkdir -p /etc/ansible \
    && echo 'localhost' > /etc/ansible/hosts

COPY ansible.cfg /etc/ansible/ansible.cfg

RUN for ansible_version in $ANSIBLE_VERSIONS; do \
        virtualenv --python=python3 ansible-$ansible_version; \
        . ansible-$ansible_version/bin/activate; \
        pip3 install ansible==$ansible_version; \
        deactivate; \
    done;

USER builder
WORKDIR /home/builder
