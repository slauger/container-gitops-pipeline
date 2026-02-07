# Configuration

Complete reference for all workflow inputs, outputs, and configuration options.

## Docker Build Workflow

### Inputs

| Input | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `image_name` | string | ✅ | - | Image name without registry prefix |
| `dockerfile` | string | ❌ | `Dockerfile` | Path to Dockerfile |
| `context` | string | ❌ | `.` | Build context path |
| `platforms` | string | ❌ | `linux/amd64` | Target platforms (comma-separated) |
| `registry` | string | ❌ | `ghcr.io` | Container registry URL |

### Secrets

| Secret | Required | Default | Description |
|--------|----------|---------|-------------|
| `REGISTRY_USERNAME` | ❌ | `github.actor` | Registry username |
| `REGISTRY_PASSWORD` | ❌ | `github.token` | Registry password or token |

Use `secrets: inherit` to pass secrets from the calling workflow.

### Outputs

| Output | Description |
|--------|-------------|
| `version` | The released version (semver on main, short sha otherwise) |
| `image` | Full image reference including registry and tag |

### Example with All Options

```yaml
jobs:
  build:
    uses: slauger/container-gitops-pipeline/.github/workflows/docker-build.yaml@v1
    with:
      image_name: my-app
      dockerfile: docker/Dockerfile.prod
      context: .
      platforms: 'linux/amd64,linux/arm64'
      registry: ghcr.io
    secrets: inherit
```

## Helm OCI Workflow

### Inputs

| Input | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `chart_path` | string | ❌ | `.` | Path to Helm chart directory |
| `registry` | string | ❌ | `ghcr.io` | OCI registry URL |

### Outputs

| Output | Description |
|--------|-------------|
| `version` | The released chart version |
| `chart` | Full chart OCI reference |

### Example with All Options

```yaml
jobs:
  build:
    uses: slauger/container-gitops-pipeline/.github/workflows/helm-oci.yaml@v1
    with:
      chart_path: charts/my-app
      registry: ghcr.io
    secrets: inherit
```

## Semantic Release

Both workflows use semantic-release with a default configuration. You can override this by adding your own config file.

### Default Configuration

```json
{
  "branches": ["main", "master"],
  "plugins": [
    ["@semantic-release/commit-analyzer", {"preset": "conventionalcommits"}],
    "@semantic-release/release-notes-generator",
    "@semantic-release/github"
  ]
}
```

### Custom Configuration

Create `.releaserc.json` in your repository root:

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
    "@semantic-release/git",
    "@semantic-release/github"
  ]
}
```

## Versioning Reference

### Docker Images

| Branch | Tag Format | Example |
|--------|------------|---------|
| `feature/*` | `:<sha>` | `:abc1234` |
| `develop` | `:<sha>` | `:abc1234` |
| `main` (no release) | `:<sha>` | `:abc1234` |
| `main` (release) | `:<semver>`, `:<major>.<minor>`, `:latest` | `:1.2.3`, `:1.2`, `:latest` |

### Helm Charts

| Branch | Version Format | Example |
|--------|----------------|---------|
| `feature/*` | `0.0.0-<sha>` | `0.0.0-abc1234` |
| `develop` | `0.0.0-<sha>` | `0.0.0-abc1234` |
| `main` (no release) | `0.0.0-<sha>` | `0.0.0-abc1234` |
| `main` (release) | `<semver>` | `1.2.3` |

## Environment Variables

The workflows use these environment variables internally:

| Variable | Source | Description |
|----------|--------|-------------|
| `GITHUB_TOKEN` | `github.token` | Authentication for registry and releases |
| `GITHUB_SHA` | Context | Full commit SHA |
| `GITHUB_REF` | Context | Git ref (branch/tag) |

No secrets need to be configured for GHCR - the workflow uses `github.token` automatically.
