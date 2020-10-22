FROM docker-rtsre-local.pruregistry.intranet.asia:8443/jenkins-agent-base:latest

USER root

ARG http_proxy
ARG https_proxy
ARG no_proxy
ARG PS_VERSION=7.0.0
ARG PS_PACKAGE=powershell-lts_${PS_VERSION}-1.debian.10_amd64.deb
ARG PS_PACKAGE_URL=https://github.com/PowerShell/PowerShell/releases/download/v${PS_VERSION}/powershell-lts_${PS_VERSION}-1.debian.10_amd64.deb

# Define ENVs for Localization/Globalization
ENV DOTNET_SYSTEM_GLOBALIZATION_INVARIANT=false \
    LC_ALL=en_US.UTF-8 \
    LANG=en_US.UTF-8 \
    # set a fixed location for the Module analysis cache
    PSModuleAnalysisCachePath=/var/cache/microsoft/powershell/PSModuleAnalysisCache/ModuleAnalysisCache \
    POWERSHELL_DISTRIBUTION_CHANNEL=PSDocker-Debian-10

RUN apt-get update \
    # Download the Linux package and save it
    && echo ${PS_PACKAGE_URL} \
    && curl -sSL -x '' ${PS_PACKAGE_URL} -o /tmp/powershell.deb \
    && apt-get install --no-install-recommends -y /tmp/powershell.deb \
    && apt-get dist-upgrade -y \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* \
    # enable en_US.UTF-8 locale
    && sed -i 's/# en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/g' /etc/locale.gen \
    # generate locale
    && locale-gen && update-locale \
    # remove powershell package
    && rm /tmp/powershell.deb \
    # intialize powershell module cache
    # and disable telemetry
    && export POWERSHELL_TELEMETRY_OPTOUT=1 \
    && pwsh \
        -NoLogo \
        -NoProfile \
        -Command " \
          \$ErrorActionPreference = 'Stop' ; \
          \$ProgressPreference = 'SilentlyContinue' ; \
          while(!(Test-Path -Path \$env:PSModuleAnalysisCachePath)) {  \
            Write-Host "'Waiting for $env:PSModuleAnalysisCachePath'" ; \
            Start-Sleep -Seconds 6 ; \
          }"

USER builder

RUN pwsh -c "Install-Module PSScriptAnalyzer -Force; Invoke-ScriptAnalyzer -ScriptDefinition 'gci'"

WORKDIR /home/builder
