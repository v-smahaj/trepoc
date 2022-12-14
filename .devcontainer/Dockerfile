# See here for image contents: https://github.com/microsoft/vscode-dev-containers/tree/v0.166.1/containers/python-3/.devcontainer/base.Dockerfile

# [Choice] Python version: 3, 3.9, 3.8, 3.7, 3.6
ARG VARIANT="3.8"
FROM mcr.microsoft.com/vscode/devcontainers/python:dev-${VARIANT}-buster

# This will be set to true when running in VSCode
ARG INTERACTIVE="false"

ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID

# Set up non-root user
COPY .devcontainer/scripts/non-root-user.sh /tmp/
SHELL ["/bin/bash", "-o", "pipefail", "-c"]
RUN bash /tmp/non-root-user.sh "${USERNAME}" "${USER_UID}" "${USER_GID}"

# Set env for tracking that we're running in a devcontainer
ENV DEVCONTAINER=true

# Install Node.js for GH actions tests and UI
ARG NODE_VERSION="lts/*"
RUN su $USERNAME -c "umask 0002 && . /usr/local/share/nvm/nvm.sh && nvm install ${NODE_VERSION} 2>&1"

# Install terraform
# version 1.3.0/1 has issues with keep recreating certificate
ARG TERRAFORM_VERSION="1.2.9"
COPY .devcontainer/scripts/terraform.sh /tmp/
RUN bash /tmp/terraform.sh "${TERRAFORM_VERSION}" /usr/bin

ARG DOCKER_GROUP_ID
COPY .devcontainer/scripts/docker-client.sh /tmp/
RUN /tmp/docker-client.sh $USERNAME

# Install Docker
RUN apt-get update && apt-get install -y ca-certificates curl gnupg lsb-release --no-install-recommends \
    && curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg \
    && echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" \
    | tee /etc/apt/sources.list.d/docker.list > /dev/null \
    && apt-get update && apt-get install -y docker-ce docker-ce-cli containerd.io --no-install-recommends \
    && apt-get clean -y && rm -rf /var/lib/apt/lists/*

# Install Certbot
RUN if [ "${INTERACTIVE}" = "true" ]; then \
    apt-get update && apt-get install -y python3 python3-venv libaugeas0 --no-install-recommends \
    && python3 -m venv /opt/certbot/ \
    && /opt/certbot/bin/pip install --no-cache-dir --upgrade pip \
    && /opt/certbot/bin/pip install --no-cache-dir certbot \
    && apt-get clean -y && rm -rf /var/lib/apt/lists/* ; fi

# Install Porter
# Not using the script from https://cdn.porter.sh/latest/install-linux.sh
# as it installs things we don't need and duplicates the binary.
ARG PORTER_MIRROR=https://cdn.porter.sh
ARG PORTER_PERMALINK=v0.38.13
ARG PORTER_PKG_PERMALINK=latest
ARG PORTER_TERRAFORM_MIXIN_PKG_PERMALINK="v1.0.0-rc.1"
ARG PORTER_HOME=/home/$USERNAME/.porter/
COPY .devcontainer/scripts/porter.sh /tmp/
RUN export PORTER_MIRROR=${PORTER_MIRROR} \
    PORTER_PERMALINK=${PORTER_PERMALINK} \
    PORTER_PKG_PERMALINK=${PORTER_PKG_PERMALINK} \
    PORTER_TERRAFORM_MIXIN_PKG_PERMALINK=${PORTER_TERRAFORM_MIXIN_PKG_PERMALINK} \
    PORTER_HOME=${PORTER_HOME} \
    && /tmp/porter.sh

ENV PATH ${PORTER_HOME}:$PATH

# Install azure-cli
ARG AZURE_CLI_VERSION=2.37.0-1~buster
COPY .devcontainer/scripts/azure-cli.sh /tmp/
RUN export AZURE_CLI_VERSION=${AZURE_CLI_VERSION} \
    && /tmp/azure-cli.sh

ARG YQ_VERSION="v4.27.2"
RUN curl -L --fail -o /usr/local/bin/yq "https://github.com/mikefarah/yq/releases/download/${YQ_VERSION}/yq_linux_amd64" \
    && chmod +x /usr/local/bin/yq

USER $USERNAME

# Save command line history
RUN echo "export HISTFILE=$HOME/commandhistory/.bash_history" >> "$HOME/.bashrc" \
    && echo "export PROMPT_COMMAND='history -a'" >> "$HOME/.bashrc" \
    && mkdir -p "$HOME/commandhistory" \
    && touch "$HOME/commandhistory/.bash_history"

# Install github-cli
COPY ./.devcontainer/scripts/gh.sh /tmp/
RUN if [ "${INTERACTIVE}" = "true" ]; then /tmp/gh.sh; fi

# Install AzureTRE OSS
ARG OSS_VERSION
ENV AZURETRE_HOME=/home/$USERNAME/AzureTRE
COPY .devcontainer/scripts/install-azure-tre-oss.sh .devcontainer/devcontainer.json /tmp/
# hadolint ignore=DL3004
RUN oss_version_in_json=$(grep -oP '(?<="OSS_VERSION": ")[^"]*' /tmp/devcontainer.json) \
    && /tmp/install-azure-tre-oss.sh "${OSS_VERSION:-$oss_version_in_json}" "${AZURETRE_HOME}" \
    && sudo chown -R $USERNAME ${AZURETRE_HOME}
