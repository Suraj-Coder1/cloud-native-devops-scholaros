FROM jenkins/jenkins:lts

USER root

RUN apt-get update \
    && apt-get install -y curl ca-certificates gnupg \
    && curl -fsSL https://deb.nodesource.com/setup_22.x | bash - \
    && apt-get install -y nodejs docker.io kubectl \
    && npm --version \
    && node --version \
    && docker --version \
    && groupadd -g 106 dockerhost \
    && usermod -aG dockerhost jenkins \
    && rm -rf /var/lib/apt/lists/*

USER jenkins
