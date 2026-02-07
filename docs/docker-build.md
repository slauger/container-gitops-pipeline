# Docker Build

The `docker-build.yaml` workflow builds and pushes Docker images with semantic versioning and multi-architecture support.

## Basic Usage

```yaml
name: Build
on:
  push:
    branches: [main, develop]

jobs:
  build:
    uses: slauger/container-gitops-pipeline/.github/workflows/docker-build.yaml@v1
    with:
      image_name: my-app
```

## Multi-Architecture Build

```yaml
jobs:
  build:
    uses: slauger/container-gitops-pipeline/.github/workflows/docker-build.yaml@v1
    with:
      image_name: my-app
      platforms: 'linux/amd64,linux/arm64'
```

See [Multi-Architecture](multi-arch.md) for details on how native runners are used.

## Inputs

| Input | Description | Default |
|-------|-------------|---------|
| `image_name` | Image name (without registry prefix) | *required* |
| `dockerfile` | Path to Dockerfile | `Dockerfile` |
| `context` | Build context path | `.` |
| `platforms` | Target platforms (comma-separated) | `linux/amd64` |
| `registry` | Container registry | `ghcr.io` |

## Outputs

| Output | Description |
|--------|-------------|
| `version` | Released version (semver or short sha) |
| `image` | Full image reference |

## Generated Tags

The workflow generates different tags depending on the branch:

### Feature/Develop Branches

```
ghcr.io/owner/my-app:abc1234              # short commit sha
ghcr.io/owner/my-app:abc1234-linux-amd64  # arch-specific (if multi-arch)
```

### Main Branch (Release)

```
ghcr.io/owner/my-app:1.2.3                # semver
ghcr.io/owner/my-app:1.2                  # major.minor
ghcr.io/owner/my-app:abc1234              # short commit sha
ghcr.io/owner/my-app:latest               # latest
ghcr.io/owner/my-app:1.2.3-linux-amd64    # arch-specific (if multi-arch)
ghcr.io/owner/my-app:latest-linux-amd64   # latest arch-specific (for GitOps)
```

## GitOps Integration

For GitOps deployments with [gitops-image-replacer](https://github.com/slauger/gitops-image-replacer), use the arch-specific tags to ensure the correct architecture is deployed:

```yaml
# values.yaml in your GitOps repo
image:
  repository: ghcr.io/owner/my-app
  tag: latest-linux-amd64  # explicit architecture
```

## Semantic Release

The workflow runs semantic-release on the `main` branch to determine the next version based on conventional commits:

| Commit Type | Version Bump |
|-------------|--------------|
| `fix:` | Patch (1.0.0 → 1.0.1) |
| `feat:` | Minor (1.0.0 → 1.1.0) |
| `feat!:` or `BREAKING CHANGE:` | Major (1.0.0 → 2.0.0) |

No `.releaserc.json` is required - a default configuration is used automatically.

## Custom Registry

To use a different registry (e.g., Docker Hub, AWS ECR):

```yaml
jobs:
  build:
    uses: slauger/container-gitops-pipeline/.github/workflows/docker-build.yaml@v1
    with:
      image_name: my-app
      registry: docker.io
    secrets:
      REGISTRY_USERNAME: ${{ secrets.DOCKER_USERNAME }}
      REGISTRY_PASSWORD: ${{ secrets.DOCKER_PASSWORD }}
```

!!! note
    For GHCR (default), no secrets are needed - the workflow uses `github.token` automatically.
