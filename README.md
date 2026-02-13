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

### gcp-wif-gcs-cdn

Authenticate via WIF, upload a local path to a GCS bucket, and invalidate Cloud CDN cache. Use for static frontend deploy (build in the workflow, then call this composite).

**Inputs:** `project_id`, `workload_identity_provider`, `service_account`, `bucket`, `path` (default: `dist`), `url_map` (optional; skip invalidation if empty).

**Usage (after building the frontend):**

```yaml
permissions:
  id-token: write
  contents: read
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci && npm run build   # or your build command
      - uses: assis-co/gh-actions-gcp/.github/actions/gcp-wif-gcs-cdn@main
        with:
          project_id: ${{ vars.GCP_PROJECT_STAGING || 'assis-staging' }}
          workload_identity_provider: ${{ vars.GCP_WIF_PROVIDER_STAGING }}
          service_account: ${{ vars.GCP_SA_STAGING }}
          bucket: ${{ vars.GCS_BUCKET_STAGING }}
          path: ${{ vars.FRONTEND_OUTPUT_DIR || 'dist' }}
          url_map: ${{ vars.CDN_URL_MAP_NAME_STAGING }}
```

**Repo variables:** `GCP_WIF_PROVIDER_STAGING`, `GCP_SA_STAGING`, `GCS_BUCKET_STAGING`, `CDN_URL_MAP_NAME_STAGING` (optional). SA needs `roles/storage.objectAdmin` on the bucket and `compute.urlMaps.invalidateCache` (or `roles/compute.loadBalancerAdmin`).

## Repo variables (per app repo)

Configure in each app repo (e.g. mono) via Settings > Variables or `gh variable set`:

- `GCP_WIF_PROVIDER_STAGING` (and `_PRODUCTION` if needed)
- `GCP_SA_STAGING` (and `_PRODUCTION` if needed)

WIF must be set up in GCP for that repo (pool + provider + SA binding).
