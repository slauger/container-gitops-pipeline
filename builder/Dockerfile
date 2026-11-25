# renovate: datasource=github-releases depName=hadolint/hadolint
ARG HADOLINT_VERSION=v2.15.1

# Node.js base image (semantic-release >= 26 requires Node.js >= 26.13,
# the Alpine nodejs package ships an older release)
FROM node:26-alpine@sha256:6bd9ab43c0ec0190ca7b2d193906cb8b7edf6e7f4afa43ddeccd6452710bffb5

# Install dependencies
RUN apk add --no-cache \
    bash \
    curl \
    git \
    jq \
    yq \
    docker-cli \
    helm

# Install hadolint (architecture-aware)
ARG HADOLINT_VERSION
ARG TARGETARCH
RUN HADOLINT_ARCH=$([ "$TARGETARCH" = "arm64" ] && echo "arm64" || echo "x86_64") && \
    curl -sSL "https://github.com/hadolint/hadolint/releases/download/${HADOLINT_VERSION}/hadolint-Linux-${HADOLINT_ARCH}" \
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
RUN node --version && \
    helm version && \
    yq --version && \
    hadolint --version && \
    semantic-release --version

WORKDIR /workspace

ENTRYPOINT ["/bin/bash"]
