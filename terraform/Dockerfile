FROM docker-rtsre-local.pruregistry.intranet.asia:8443/jenkins-agent-base:latest

USER root

ARG TERRAFORM_DOCS_VERSION="v0.10.1"
ARG TFLINT_VERSION="v0.20.2"
ARG TERRAFORM_MINOR_VERSIONS="0.11 0.12 0.13"

ENV ARM_PARTNER_ID="3492932"

RUN git clone --depth 1 https://github.com/tfutils/tfenv.git /opt/tfenv \
    && ln -s /opt/tfenv/bin/tfenv /usr/local/bin \
    && ln -s /opt/tfenv/bin/terraform /usr/local/bin \
    && mkdir -p /opt/tfenv/versions \
    && touch /opt/tfenv/version \
    && chmod -R a+w /opt/tfenv \
    && curl -L https://github.com/terraform-linters/tflint/releases/download/${TFLINT_VERSION}/tflint_linux_amd64.zip -o tflint.zip && unzip tflint.zip -d /usr/local/bin && rm tflint.zip \
    && curl -Lo /usr/local/bin/terraform-docs https://github.com/segmentio/terraform-docs/releases/download/${TERRAFORM_DOCS_VERSION}/terraform-docs-${TERRAFORM_DOCS_VERSION}-linux-amd64 \
    && chmod +x /usr/local/bin/terraform-docs \
    && cd /opt

RUN for terraform_minor_version in $TERRAFORM_MINOR_VERSIONS; do \
        tfenv install latest:$terraform_minor_version; \
    done;

USER builder
WORKDIR /home/builder
