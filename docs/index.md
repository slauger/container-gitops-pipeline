# Container GitOps Pipeline

Reusable GitHub Actions workflows for container-based GitOps pipelines. Build Docker images and Helm charts with semantic versioning, multi-arch support, and automated GitOps deployment.

## Features

- 🏷️ **Semantic Versioning** - Automated releases with [semantic-release](https://github.com/semantic-release/semantic-release)
- 🖥️ **Multi-Architecture** - Native amd64 and arm64 builds via GitHub runners (no QEMU)
- 🔄 **GitOps Ready** - Works with [gitops-image-replacer](https://github.com/slauger/gitops-image-replacer) for ArgoCD deployments
- 📦 **OCI Registry** - Push Docker images and Helm charts to any OCI-compliant registry
- ⚡ **Zero Config** - Sensible defaults, no `.releaserc.json` required
- 📌 **Pinned Dependencies** - All tools versioned and managed via Renovate

## How it works

### Docker Image Versioning

```mermaid
flowchart LR
    subgraph Branches
        feature["feature/*"]
        develop["develop"]
        main["main"]
    end

    subgraph Tags
        sha[":abc1234"]
        semver[":1.2.3"]
        minor[":1.2"]
        latest[":latest"]
    end

    feature --> sha
    develop --> sha
    main -->|"release"| semver
    main -->|"release"| minor
    main -->|"release"| latest

    classDef shaStyle fill:#1e88e5,stroke:#1565c0,color:#fff
    classDef semverStyle fill:#43a047,stroke:#2e7d32,color:#fff
    classDef latestStyle fill:#fb8c00,stroke:#ef6c00,color:#fff

    class sha shaStyle
    class semver,minor semverStyle
    class latest latestStyle
```

### Helm Chart Versioning

```mermaid
flowchart LR
    subgraph Branches
        feature["feature/*"]
        develop["develop"]
        main["main"]
    end

    subgraph Versions
        pre["0.0.0-abc1234"]
        semver["1.2.3"]
    end

    feature --> pre
    develop --> pre
    main -->|"release"| semver

    classDef preStyle fill:#7b1fa2,stroke:#6a1b9a,color:#fff
    classDef semverStyle fill:#43a047,stroke:#2e7d32,color:#fff

    class pre preStyle
    class semver semverStyle
```

### GitOps Deployment

```mermaid
flowchart LR
    subgraph Artifacts
        image["Docker Image<br/>:2.1.0"]
        chart["Helm Chart<br/>1.5.3"]
    end

    subgraph GitOps["GitOps Repository"]
        values["values.yaml<br/><i>image: :2.1.0</i>"]
        appCR["Application CR<br/><i>chart: 1.5.3</i>"]
    end

    subgraph Cluster["Kubernetes"]
        argocd["ArgoCD"]
        app["Deployment"]
    end

    image -->|"gitops-image-replacer"| values
    chart -->|"gitops-image-replacer"| appCR
    values --> argocd
    appCR --> argocd
    argocd -->|"sync"| app

    classDef artifactStyle fill:#1e88e5,stroke:#1565c0,color:#fff
    classDef gitopsStyle fill:#fb8c00,stroke:#ef6c00,color:#fff
    classDef clusterStyle fill:#43a047,stroke:#2e7d32,color:#fff

    class image,chart artifactStyle
    class values,appCR gitopsStyle
    class argocd,app clusterStyle
```

## Quick Start

=== "Docker Images"

    Add `.github/workflows/build.yaml` to your repository:

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

=== "Helm Charts"

    Add `.github/workflows/release.yaml` to your repository:

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

That's it! Your images/charts will automatically build and push on every commit.

## Next Steps

- [Docker Build](docker-build.md) - Build and push container images
- [Helm OCI](helm-oci.md) - Package and push Helm charts
- [Multi-Architecture](multi-arch.md) - Native amd64 and arm64 builds
- [Configuration](configuration.md) - All workflow inputs and outputs
