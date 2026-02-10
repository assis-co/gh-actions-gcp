# gh-actions-gcp

Shared GitHub Actions for GCP (GKE) with auth via Workload Identity Federation. Reusable by multiple apps (mono, backend-api, etc.).

## Actions

### gcp-wif-gke

Authenticate to GCP via WIF and configure kubectl for a GKE cluster.

**Inputs:** `project_id`, `cluster_name`, `region`, `workload_identity_provider`, `service_account`

**Usage (in any repo with WIF variables configured):**

```yaml
permissions:
  id-token: write
  contents: read
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5
      - uses: assis-co/gh-actions-gcp/.github/actions/gcp-wif-gke@main
        with:
          project_id: assis-staging
          cluster_name: gke-blue-staging
          region: us-central1
          workload_identity_provider: ${{ vars.GCP_WIF_PROVIDER_STAGING }}
          service_account: ${{ vars.GCP_SA_STAGING }}
      # then your app steps: kubectl apply, rollouts set image, etc.
```

### gcp-wif-gke-rollouts

Same as above plus installs `kubectl-argo-rollouts`. Use for apps deployed via Argo Rollouts.

**Inputs:** same as gcp-wif-gke, plus optional `rollouts_version` (default 1.8.3).

**Usage:**

```yaml
- uses: assis-co/gh-actions-gcp/.github/actions/gcp-wif-gke-rollouts@main
  with:
    project_id: assis-staging
    cluster_name: gke-blue-staging
    region: us-central1
    workload_identity_provider: ${{ vars.GCP_WIF_PROVIDER_STAGING }}
    service_account: ${{ vars.GCP_SA_STAGING }}
- run: kubectl argo rollouts set image my-app *=${{ env.IMAGE }} -n my-namespace
- run: kubectl argo rollouts status my-app -n my-namespace
```

## Repo variables (per app repo)

Configure in each app repo (e.g. mono) via Settings > Variables or `gh variable set`:

- `GCP_WIF_PROVIDER_STAGING` (and `_PRODUCTION` if needed)
- `GCP_SA_STAGING` (and `_PRODUCTION` if needed)

WIF must be set up in GCP for that repo (pool + provider + SA binding).
