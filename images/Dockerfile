# Source: https://github.com/dotnet/dotnet-docker
FROM mcr.microsoft.com/dotnet/runtime-deps:8.0-noble as build

ARG TARGETOS
ARG TARGETARCH
ARG RUNNER_VERSION
ARG RUNNER_CONTAINER_HOOKS_VERSION=0.6.0
ARG DOCKER_VERSION=25.0.5
ARG BUILDX_VERSION=0.14.1

RUN apt update -y && apt install curl unzip -y

WORKDIR /actions-runner
RUN export RUNNER_ARCH=${TARGETARCH} \
    && if [ "$RUNNER_ARCH" = "amd64" ]; then export RUNNER_ARCH=x64 ; fi \
    && curl -f -L -o runner.tar.gz https://github.com/dfinity/runner/releases/download/v${RUNNER_VERSION}/actions-runner-${TARGETOS}-${RUNNER_ARCH}-${RUNNER_VERSION}.tar.gz \
    && tar xzf ./runner.tar.gz \
    && rm runner.tar.gz

RUN curl -f -L -o runner-container-hooks.zip https://github.com/actions/runner-container-hooks/releases/download/v${RUNNER_CONTAINER_HOOKS_VERSION}/actions-runner-hooks-k8s-${RUNNER_CONTAINER_HOOKS_VERSION}.zip \
    && unzip ./runner-container-hooks.zip -d ./k8s \
    && rm runner-container-hooks.zip

FROM mcr.microsoft.com/dotnet/runtime-deps:8.0-noble

ENV DEBIAN_FRONTEND=noninteractive
ENV RUNNER_MANUALLY_TRAP_SIG=1
ENV ACTIONS_RUNNER_PRINT_LOG_TO_STDOUT=1
ENV ImageOS=ubuntu24

# 'gpg-agent' and 'software-properties-common' are needed for the 'add-apt-repository' command that follows
RUN apt update -y \
    && apt install -y --no-install-recommends sudo lsb-release gpg-agent software-properties-common podman \
    && rm -rf /var/lib/apt/lists/*

# Configure git-core/ppa based on guidance here:  https://git-scm.com/download/linux
RUN add-apt-repository ppa:git-core/ppa \
    && apt update -y

RUN adduser --disabled-password --gecos "" --uid 1001 runner \
    && usermod -aG sudo runner \
    && echo "%sudo   ALL=(ALL:ALL) NOPASSWD:ALL" > /etc/sudoers \
    && echo "Defaults env_keep += \"DEBIAN_FRONTEND\"" >> /etc/sudoers

COPY <<EOF /usr/bin/docker
#!/usr/bin/env bash
sudo podman "$@"
EOF
RUN chmod +x /usr/bin/docker

WORKDIR /home/runner

COPY --chown=runner:runner --from=build /actions-runner .

USER runner
