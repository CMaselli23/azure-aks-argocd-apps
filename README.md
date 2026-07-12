# azure-aks-argocd-apps

GitOps source of truth for applications running on the `aks-argocd-lab` cluster.

Argo CD watches this repository and reconciles the cluster to match its declared
state. Nothing here is deployed by hand — every workload arrives via Git commit.

Infrastructure (Terraform) lives separately in
[azure-aks-argocd](https://github.com/CMaselli23/azure-aks-argocd). Application
manifests and IaC are deliberately split: they have different change frequencies,
different blast radii, and different access-control requirements.

---

## Structure

├── app-of-apps.yaml        # Root Application — watches applications/
├── applications/           # Argo CD Application definitions (the children)
│   ├── demo-app.yaml
│   └── monitoring.yaml
└── apps/                   # Workload manifests the children point at
└── demo-app/
├── deployment.yaml
└── service.yaml

## App-of-apps pattern

`app-of-apps.yaml` is a single parent Application that watches `applications/`.
Every Application manifest found there is created and managed automatically.

Applying the parent once bootstraps the entire platform:

    kubectl apply -f app-of-apps.yaml

Adding a new application to the cluster requires no `kubectl` at all — commit a
new Application manifest to `applications/` and the parent reconciles it in.

This makes the platform's composition a single declarative source: `applications/`
**is** the authoritative list of everything running. Adding a workload becomes a
reviewable pull request rather than a manual apply — deployment inherits code
review, audit history, and revertibility.

## Managed applications

| Application | Source | Purpose |
|---|---|---|
| `demo-app` | `apps/demo-app` (this repo) | Sample workload proving Git-to-cluster reconciliation |
| `monitoring` | `kube-prometheus-stack` Helm chart | Prometheus, Grafana, Alertmanager — observability stack |

The monitoring stack is declared as a pinned Helm chart version rather than
installed imperatively. No credentials appear in any manifest — Grafana generates
its admin password into a Kubernetes Secret at install time.

## Sync policy

All Applications run with:

- **`selfHeal: true`** — drift introduced directly against the cluster is
  automatically reverted to match Git
- **`prune: true`** — resources removed from Git are removed from the cluster
- **`resources-finalizer`** — deleting an Application cascades to the workloads
  it deployed, preventing orphans

Part of the Maselli Technologies SRE Training Curriculum — Month 4.