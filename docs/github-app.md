# GitHub App Setup

The pipeline can use a GitHub App to update references in your GitOps repository via [gitops-image-replacer](https://github.com/slauger/gitops-image-replacer) (for Docker images) or [gitops-replacer](https://github.com/slauger/gitops-replacer) (for Helm charts). This is required because the pipeline needs write access to a different repository than the one it's running in.

## Why a GitHub App?

| Method | Cross-Repo Write | Token Expiry | Granular Permissions |
|--------|------------------|--------------|---------------------|
| `GITHUB_TOKEN` | No | Per-job | No |
| Personal Access Token (PAT) | Yes | Manual | No |
| **GitHub App** | **Yes** | **Auto (1h)** | **Yes** |

GitHub Apps are the recommended approach because:

- Tokens expire automatically (1 hour)
- Fine-grained permissions per repository
- Not tied to a personal account
- Audit log shows app actions separately

## Step 1: Create the GitHub App

1. Go to **GitHub Settings** → **Developer settings** → **GitHub Apps** → **New GitHub App**

   Or use this direct link: [https://github.com/settings/apps/new](https://github.com/settings/apps/new)

2. Fill in the basic information:

   | Field | Value |
   |-------|-------|
   | **GitHub App name** | `gitops-deployer` (or your choice) |
   | **Homepage URL** | Your repository URL |
   | **Webhook** | Uncheck "Active" (not needed) |

3. Set **Repository permissions**:

   | Permission | Access |
   |------------|--------|
   | **Contents** | Read and write |

   This is the only permission needed. The app will commit image reference updates to your GitOps repository.

4. Set **Where can this GitHub App be installed?**:

   - Select **Only on this account** (recommended)

5. Click **Create GitHub App**

## Step 2: Note the App ID

After creation, you'll see the App settings page.

1. Find the **App ID** (a number like `123456`)
2. Save this as the `GITOPS_APP_ID` secret in your image repository

## Step 3: Generate a Private Key

1. Scroll down to **Private keys**
2. Click **Generate a private key**
3. A `.pem` file will be downloaded
4. Save the **entire contents** of this file as the `GITOPS_APP_PRIVATE_KEY` secret

The private key looks like this:

```
-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEA...
...
-----END RSA PRIVATE KEY-----
```

## Step 4: Install the App

The app must be installed on the GitOps repository to grant access.

1. Go to **Install App** in the left sidebar
2. Click **Install** next to your account/organization
3. Select **Only select repositories**
4. Choose your **GitOps repository** (e.g., `myorg/gitops`)
5. Click **Install**

## Step 5: Configure Secrets

Add these secrets to your image repository (not the GitOps repo):

| Secret | Value |
|--------|-------|
| `GITOPS_APP_ID` | The App ID from Step 2 |
| `GITOPS_APP_PRIVATE_KEY` | The entire `.pem` file contents from Step 3 |

### Adding secrets via GitHub UI

1. Go to your image repository → **Settings** → **Secrets and variables** → **Actions**
2. Click **New repository secret**
3. Add `GITOPS_APP_ID` with the App ID
4. Add `GITOPS_APP_PRIVATE_KEY` with the private key contents

### Adding secrets via GitHub CLI

```bash
gh secret set GITOPS_APP_ID --body "123456"
gh secret set GITOPS_APP_PRIVATE_KEY < path/to/private-key.pem
```

## Usage with gitops-image-replacer (Docker Images)

After the Docker build completes, use gitops-image-replacer to update image references in your GitOps repository:

```yaml
jobs:
  build:
    uses: slauger/container-gitops-pipeline/.github/workflows/docker-build.yaml@v1
    with:
      image_name: my-app

  update-gitops:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Generate token
        id: app-token
        uses: actions/create-github-app-token@v1
        with:
          app-id: ${{ secrets.GITOPS_APP_ID }}
          private-key: ${{ secrets.GITOPS_APP_PRIVATE_KEY }}
          owner: ${{ github.repository_owner }}
          repositories: gitops

      - name: Update GitOps
        uses: slauger/gitops-image-replacer@v1
        with:
          token: ${{ steps.app-token.outputs.token }}
          repository: myorg/gitops
          file: apps/my-app/values.yaml
          image: ${{ needs.build.outputs.image }}
```

## Usage with gitops-replacer (Helm Charts)

After the Helm chart build completes, use gitops-replacer to update chart version references in your GitOps repository.

First, add a marker comment in your target file (e.g., `Chart.yaml`):

```yaml
dependencies:
  # gitops-replacer: my-chart
  - name: my-chart
    version: "0.0.0-abc1234"
    repository: oci://ghcr.io/myorg
```

Then configure the workflow:

```yaml
jobs:
  build:
    uses: slauger/container-gitops-pipeline/.github/workflows/helm-oci.yaml@v1
    with:
      chart_path: '.'

  update-gitops:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Generate token
        id: app-token
        uses: actions/create-github-app-token@v1
        with:
          app-id: ${{ secrets.GITOPS_APP_ID }}
          private-key: ${{ secrets.GITOPS_APP_PRIVATE_KEY }}
          owner: ${{ github.repository_owner }}
          repositories: gitops

      - name: Install gitops-replacer
        run: pip install gitops-replacer

      - name: Update GitOps
        env:
          GITHUB_TOKEN: ${{ steps.app-token.outputs.token }}
        run: |
          gitops-replacer --apply "${{ needs.build.outputs.version }}"
```

The `gitops-replacer.json` config in your chart repository:

```json
{
  "gitops-replacer": [
    {
      "repository": "myorg/gitops",
      "branch": "main",
      "file": "apps/my-app/Chart.yaml",
      "depName": "my-chart",
      "when": "^refs/heads/main$"
    }
  ]
}
```

## Verification

After setup, the pipeline will:

1. Generate a short-lived token using the App credentials
2. Use this token to push commits to your GitOps repository
3. The token expires after 1 hour automatically

You can verify the setup by checking:

- **GitOps repository** → **Settings** → **Integrations** → **GitHub Apps**
- Your app should be listed with "Contents: Read and write"

## Troubleshooting

### "Resource not accessible by integration"

The app is not installed on the GitOps repository. Go to the app settings and install it on the correct repository.

### "Bad credentials"

The private key is incorrect or malformed. Make sure you copied the entire `.pem` file including the `-----BEGIN RSA PRIVATE KEY-----` and `-----END RSA PRIVATE KEY-----` lines.

### "Integration not found"

The App ID is incorrect. Double-check the App ID in your GitHub App settings.

## Multiple GitOps Repositories

If you deploy to multiple GitOps repositories (e.g., different environments in different repos), install the app on all of them:

1. Go to your GitHub App → **Install App**
2. Select **Only select repositories**
3. Add all GitOps repositories that the pipeline needs to update

## See Also

- [Docker Build](docker-build.md)
- [Helm OCI](helm-oci.md)
- [Configuration](configuration.md)
- [gitops-image-replacer](https://github.com/slauger/gitops-image-replacer) - For Docker images
- [gitops-replacer](https://github.com/slauger/gitops-replacer) - For Helm charts
- [GitHub Docs: Creating a GitHub App](https://docs.github.com/en/apps/creating-github-apps)
