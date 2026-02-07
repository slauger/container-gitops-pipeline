FROM alpine:3.19

# Install dependencies
RUN apk add --no-cache \
    bash \
    curl \
    git \
    jq \
    yq \
    nodejs \
    npm \
    docker-cli \
    helm

# Install hadolint
RUN curl -sSL https://github.com/hadolint/hadolint/releases/download/v2.12.0/hadolint-Linux-x86_64 \
    -o /usr/local/bin/hadolint && \
    chmod +x /usr/local/bin/hadolint

# Install semantic-release
RUN npm install -g \
    semantic-release \
    @semantic-release/git \
    @semantic-release/changelog \
    @semantic-release/exec

# Verify installations
RUN helm version && \
    yq --version && \
    hadolint --version && \
    semantic-release --version

WORKDIR /workspace

ENTRYPOINT ["/bin/bash"]
