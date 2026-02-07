# Builder Image

A pre-built Docker image with all tools needed for CI/CD pipelines.

## Image

```
ghcr.io/slauger/container-gitops-pipeline/builder:latest
```

## Included Tools

| Tool | Purpose |
|------|---------|
| Docker CLI | Building and pushing images |
| Helm | Packaging and pushing charts |
| hadolint | Dockerfile linting |
| yq | YAML processing |
| jq | JSON processing |
| semantic-release | Automated versioning |
| Node.js / npm | JavaScript runtime |
| git | Version control |
| curl | HTTP requests |
| bash | Shell scripting |

## Usage

### GitHub Actions

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    container:
      image: ghcr.io/slauger/container-gitops-pipeline/builder:latest
    steps:
      - uses: actions/checkout@v4
      - run: hadolint Dockerfile
      - run: helm lint ./chart
```

### Local Development

```bash
docker run -it --rm \
  -v $(pwd):/workspace \
  -w /workspace \
  ghcr.io/slauger/container-gitops-pipeline/builder:latest \
  hadolint Dockerfile
```

## Version Pinning

All tools in the builder image are version-pinned and updated via Renovate:

```dockerfile
# renovate: datasource=docker depName=alpine
ARG ALPINE_VERSION=3.19

# renovate: datasource=github-releases depName=hadolint/hadolint
ARG HADOLINT_VERSION=v2.12.0
```

## Semantic Release Plugins

The following semantic-release plugins are pre-installed:

- `@semantic-release/commit-analyzer`
- `@semantic-release/release-notes-generator`
- `@semantic-release/github`
- `@semantic-release/git`
- `@semantic-release/changelog`
- `@semantic-release/exec`
- `conventional-changelog-conventionalcommits`

## Building Locally

```bash
cd builder
docker build -t builder:local .
```
