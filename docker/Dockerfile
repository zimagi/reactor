#
#====================================================================
# Reactor Docker image
#
#
# Base image
#
ARG REACTOR_PARENT_IMAGE="ubuntu:22.04"
FROM ${REACTOR_PARENT_IMAGE}
#
# Computer Processing Chip architecture
#
ARG REACTOR_ARCHITECTURE="amd64"
#
# Dockerfile arguments
#
ARG REACTOR_USER_NAME=reactor
ARG REACTOR_USER_UID=1000
ARG REACTOR_DOCKER_GID=135
#
#====================================================================
# Core system configuration
#
#
# Core environment variables
#
ENV LANG=C.UTF-8
ENV PYTHONUNBUFFERED=1
ENV REQUESTS_CA_BUNDLE=/etc/ssl/certs/ca-certificates.crt
#
# Shell environment
#
SHELL ["/bin/bash", "--login", "-c"]
RUN echo 'debconf debconf/frontend select Noninteractive' | debconf-set-selections
#
# Package repository management
#
COPY ./docker/packages.core.txt /root/packages.core.txt
RUN apt-get update -y \
  && apt-get upgrade -y \
  && sed '/^\s*\#.*$/d' /root/packages.core.txt | xargs -r apt-get install -y --no-install-recommends \
  && rm -rf /var/lib/apt/lists/*

RUN curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg \
  && echo "deb [signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] \
  https://download.docker.com/linux/ubuntu/ $(lsb_release -cs) stable" > /etc/apt/sources.list.d/docker.list
#
# System dependencies
#
COPY ./docker/packages.app.txt /root/packages.app.txt
RUN apt-get update -y \
  && sed '/^\s*\#.*$/d' /root/packages.app.txt | xargs -r apt-get install -y --no-install-recommends \
  && rm -rf /var/lib/apt/lists/*
#
# Reactor core Python packages
#
COPY ./requirements.txt /root/requirements.txt
RUN pip3 install --no-cache-dir -r /root/requirements.txt
#
# User initialization
#
ENV HOME_DIR=/home/${REACTOR_USER_NAME}

RUN groupmod --gid ${REACTOR_DOCKER_GID} docker \
  && groupadd -f --system --gid ${REACTOR_USER_UID} ${REACTOR_USER_NAME} \
  && useradd --system --create-home \
  --home-dir ${HOME_DIR} \
  --shell /bin/bash \
  --uid ${REACTOR_USER_UID} \
  --gid ${REACTOR_USER_NAME} \
  --groups sudo,docker \
  ${REACTOR_USER_NAME} \
  && echo '%sudo ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers \
  && echo 'alias reactor="reactor-exec"' >> /etc/bash.bashrc
#
#====================================================================
# Application configuration
#
#
# Application environment variables
#
ENV REACTOR_DIR="/reactor"
ENV SCRIPT_DIR="${REACTOR_DIR}/scripts"
ENV SHARE_DIR="${REACTOR_DIR}/share"

ENV PATH="${SCRIPT_DIR}:${PATH}"
#
# Reactor directory
#
COPY --chown=${REACTOR_USER_NAME}:${REACTOR_USER_NAME} ./ ${REACTOR_DIR}
VOLUME ${REACTOR_DIR}

RUN ln -s ${REACTOR_DIR}/VERSION /REACTOR.txt
#
# Project customization
#
RUN test ! -f ${SHARE_DIR}/requirements.txt \
  || pip3 install --no-cache-dir -r ${SHARE_DIR}/requirements.txt

RUN test ! -x ${SHARE_DIR}/install.sh \
  || ${SHARE_DIR}/install.sh
#
# Execution gateway
#
USER ${REACTOR_USER_NAME}
WORKDIR ${HOME_DIR}
ENTRYPOINT ["reactor-exec"]
