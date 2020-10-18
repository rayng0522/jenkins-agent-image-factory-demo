FROM prudential/generic-base:latest

USER root

ARG ANSIBLE_VERSION=2.10.0
ARG ANSIBLE_LINT_VERSION=4.2.0

RUN apt-get update \
    && apt-get dist-upgrade -y \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* \
    && pip3 --no-cache-dir install \
        cffi \
        ansible==${ANSIBLE_VERSION} \
        ansible-lint==${ANSIBLE_LINT_VERSION} \
        pywinrm

RUN mkdir /ansible \
    && mkdir -p /etc/ansible \
    && echo 'localhost' > /etc/ansible/hosts

COPY ansible.cfg /etc/ansible/ansible.cfg

USER builder
WORKDIR /home/builder
