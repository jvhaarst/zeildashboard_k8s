# zeildashboard_k8s

Helm chart for the **Rhine sailing-conditions dashboard**, published as a Helm
repository via GitHub Pages so it can be added as a chart repository in Rancher.

- **Live site:** https://zeilweer.vanhaarst.net
- **Image:** `ghcr.io/jvhaarst/zeildashboard` (built from
  [jvhaarst/zeildashboard](https://github.com/jvhaarst/zeildashboard))

## How updates flow

```
php:8.3-apache release ──▶ Renovate (zeildashboard repo) bumps Dockerfile
        │                          │
        │                          ▼
 dashboard code change ───▶ new image  ghcr.io/jvhaarst/zeildashboard:1.4.N
                                       │
                                       ▼
                 Renovate (this repo) bumps Chart.yaml appVersion
                                       │
                                       ▼
              helm-release workflow publishes a new chart version
                                       │
                                       ▼
                       Rancher sees the new chart version
```

- `renovate.json` watches the `ghcr.io/jvhaarst/zeildashboard` tag in
  `zeildashboard/Chart.yaml` (`appVersion`) via the Docker datasource and opens a
  PR when a newer image is published.
- Merging to `main` (or any change under `zeildashboard/`) triggers
  `.github/workflows/helm-release.yaml`, which packages the chart
  (version auto-set to `<base>.<run_number>`) and publishes it + an updated
  `index.yaml` to GitHub Pages.

## Use it

Add the repo in Rancher (Apps → Repositories) or with the Helm CLI:

```bash
helm repo add zeildashboard https://jvhaarst.github.io/zeildashboard_k8s
helm repo update
helm upgrade --install zeilweer zeildashboard/zeildashboard \
  --namespace zeil-dashboard --create-namespace
```

## Configuration

| Value | Default | Notes |
|-------|---------|-------|
| `image.repository` | `ghcr.io/jvhaarst/zeildashboard` | |
| `image.tag` | `""` | Empty → uses the chart `appVersion` (Renovate-managed). |
| `ingress.host` | `zeilweer.vanhaarst.net` | Needs a public DNS record for Let's Encrypt. |
| `ingress.clusterIssuer` | `letsencrypt-production` | cert-manager ClusterIssuer. |
| `language` | `nl` | UI language (`RSC_LANG`). |
| `replicaCount` | `1` | |

## Local development

```bash
helm lint zeildashboard
helm template zeilweer zeildashboard --namespace zeil-dashboard
```
