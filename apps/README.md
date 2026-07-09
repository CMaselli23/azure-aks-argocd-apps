# azure-aks-argocd-apps

Kubernetes application manifests reconciled to AKS by Argo CD.

This repository is the **GitOps source of truth** for application workloads.
It is deliberately separate from the infrastructure (Terraform) repository
`azure-aks-argocd`, following the principle that application manifests and
infrastructure code have different change frequencies, risk profiles, and
access-control requirements.

Argo CD watches this repo and reconciles the cluster to match its declared state.

Part of the Maselli Technologies SRE Training Curriculum — Month 4: GitOps with Argo CD.