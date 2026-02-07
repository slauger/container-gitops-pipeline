# renovate: datasource=docker depName=alpine
ARG ALPINE_VERSION=3.19

# renovate: datasource=github-releases depName=hadolint/hadolint
ARG HADOLINT_VERSION=v2.12.0

FROM alpine:${ALPINE_VERSION}

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
ARG HADOLINT_VERSION
RUN curl -sSL https://github.com/hadolint/hadolint/releases/download/${HADOLINT_VERSION}/hadolint-Linux-x86_64 \
    -o /usr/local/bin/hadolint && \
    chmod +x /usr/local/bin/hadolint

# Install semantic-release (pinned versions via npm ci)
WORKDIR /opt/semantic-release
COPY package.json package-lock.json ./
RUN npm ci && npm cache clean --force
ENV PATH="/opt/semantic-release/node_modules/.bin:${PATH}"

# Default semantic-release config (used if no .releaserc.json in repo)
COPY default-releaserc.json /etc/semantic-release/default-releaserc.json

# Verify installations
RUN helm version && \
    yq --version && \
    hadolint --version && \
    semantic-release --version

WORKDIR /workspace

ENTRYPOINT ["/bin/bash"]
