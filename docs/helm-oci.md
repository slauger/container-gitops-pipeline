# Helm OCI

The `helm-oci.yaml` workflow packages and pushes Helm charts to an OCI registry with semantic versioning.

## Basic Usage

```yaml
name: Release
on:
  push:
    branches: [main, develop]

jobs:
  release:
    uses: slauger/container-gitops-pipeline/.github/workflows/helm-oci.yaml@v1
    with:
      chart_path: '.'
```

## Inputs

| Input | Description | Default |
|-------|-------------|---------|
| `chart_path` | Path to Helm chart directory | `.` |
| `registry` | OCI registry | `ghcr.io` |

## Outputs

| Output | Description |
|--------|-------------|
| `version` | Released chart version |
| `chart` | Full chart OCI reference |

## Versioning

The workflow automatically updates the `version` field in `Chart.yaml`:

| Branch | Version Format | Example |
|--------|----------------|---------|
| `feature/*` | `0.0.0-<sha>` | `0.0.0-abc1234` |
| `develop` | `0.0.0-<sha>` | `0.0.0-abc1234` |
| `main` | Semver | `1.2.3` |

## Chart Repository Structure

```
my-chart/
├── Chart.yaml
├── values.yaml
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── ...
└── .github/
    └── workflows/
        └── release.yaml
```

## Using the Chart

After the workflow pushes the chart, you can use it with Helm:

```bash
# Login to registry (if private)
helm registry login ghcr.io -u USERNAME -p TOKEN

# Pull the chart
helm pull oci://ghcr.io/owner/my-chart --version 1.2.3

# Install directly
helm install my-release oci://ghcr.io/owner/my-chart --version 1.2.3
```

## ArgoCD Integration

To use OCI charts with ArgoCD:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
spec:
  source:
    repoURL: ghcr.io/owner
    chart: my-chart
    targetRevision: 1.2.3
```

## Semantic Release

Like the Docker workflow, semantic-release determines versions on the `main` branch:

| Commit Type | Version Bump |
|-------------|--------------|
| `fix:` | Patch (1.0.0 → 1.0.1) |
| `feat:` | Minor (1.0.0 → 1.1.0) |
| `feat!:` or `BREAKING CHANGE:` | Major (1.0.0 → 2.0.0) |

No `.releaserc.json` is required.
