# container-gitops-pipeline

Reusable GitHub Actions workflows for container-based GitOps pipelines.

## Workflows

### Docker Build (`docker-build.yaml`)

Builds and pushes Docker images to an OCI registry with semantic versioning.

**Features:**
- Semantic Release for versioning
- Multi-platform builds (amd64, arm64)
- GHCR integration
- Branch-based tagging (`:develop`, `:main`, `:v1.2.3`)

**Usage:**

```yaml
# .github/workflows/build.yaml
name: Build

on:
  push:
    branches: [main, develop]

jobs:
  build:
    uses: slauger/container-gitops-pipeline/.github/workflows/docker-build.yaml@main
    with:
      image_name: my-app
    secrets:
      GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

**Inputs:**

| Input | Description | Default |
|-------|-------------|---------|
| `image_name` | Image name (without registry) | required |
| `dockerfile` | Path to Dockerfile | `Dockerfile` |
| `context` | Build context | `.` |
| `platforms` | Target platforms | `linux/amd64` |
| `registry` | Container registry | `ghcr.io` |

---

### Helm OCI (`helm-oci.yaml`)

Packages and pushes Helm charts to an OCI registry with semantic versioning.

**Features:**
- Semantic Release for versioning
- Automatic `Chart.yaml` version update
- OCI registry support
- Develop branch pre-releases

**Usage:**

```yaml
# .github/workflows/release.yaml
name: Release

on:
  push:
    branches: [main, develop]

jobs:
  release:
    uses: slauger/container-gitops-pipeline/.github/workflows/helm-oci.yaml@main
    with:
      chart_path: '.'
    secrets:
      GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

**Inputs:**

| Input | Description | Default |
|-------|-------------|---------|
| `chart_path` | Path to Helm chart | `.` |
| `registry` | OCI registry | `ghcr.io` |

---

## Repository Structure

Recommended structure for projects using these workflows:

```
# Image Repository (eibtalerhof-image-mail-api)
├── Dockerfile
├── src/
├── .releaserc.json
└── .github/workflows/
    └── build.yaml          # calls docker-build.yaml

# Chart Repository (eibtalerhof-chart-mail-api)
├── Chart.yaml
├── values.yaml
├── templates/
├── .releaserc.json
└── .github/workflows/
    └── release.yaml        # calls helm-oci.yaml
```

## Semantic Release Config

Add `.releaserc.json` to your repository:

```json
{
  "branches": [
    "main",
    {"name": "develop", "prerelease": true}
  ],
  "plugins": [
    "@semantic-release/commit-analyzer",
    "@semantic-release/release-notes-generator",
    "@semantic-release/changelog",
    "@semantic-release/git"
  ]
}
```

## Versioning Strategy

| Branch | Image Tag | Helm Version |
|--------|-----------|--------------|
| `develop` | `:develop` | `0.0.0-develop.123` |
| `main` | `:v1.2.3`, `:latest` | `1.2.3` |

## Builder Image

A builder image is provided with pre-installed tools:

- Docker CLI
- Helm
- yq
- semantic-release

Pull from: `ghcr.io/slauger/container-gitops-pipeline/builder:latest`

## License

MIT
