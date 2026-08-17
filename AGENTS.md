# CLAUDE.md

This file provides guidance to AI Agents when working with code in this repository.

## What this repo is

This is the **public** half of Sigurs' GitOps repo (`github.com/Sigurs/gitops`), deployed by ArgoCD onto two k3s clusters. There is no application source code, build step, or test suite here — it's entirely Kubernetes manifests (plain YAML + Kustomize) and ArgoCD `Application` resources. A companion **private** repo (`gitops-private`, referenced via SSH as `ssh://git@github.com/Sigurs/gitops-private.git`) holds resources that shouldn't be public and is wired in as a nested ArgoCD app (see `home-k8s/argocd-apps/admin/private-argocd-apps.yaml`). There's no way to validate secret-bearing changes from this repo alone.

There is no lint/build/test command — the closest thing to validation is running `kubectl kustomize <dir>` (or `kustomize build`) against a changed `kustomization.yaml` to confirm it renders, and `kubectl apply --dry-run=client -k <dir>` if a live cluster context is configured.

## Clusters and top-level layout

Each top-level directory under this repo corresponds to one k3s cluster and is deployed independently:

- `home-k8s/` — cluster running at home (Raspberry Pi + Proxmox VM), no public access. Uses UserNamespaces for containers needing root, since NFS-backed storage doesn't support UserNamespaces cleanly.
- `proxima/` — cluster in Hetzner Cloud. Uses Cilium in native-routing mode over Hetzner Cloud Networks, and Cloudflared (not paid Hetzner LBs) for ingress.

Within each cluster directory, the convention is consistent:

- `namespaces/` — one `Namespace` YAML per app/component, split into `admin/` (cluster infra) and `apps/` (workloads), aggregated by a `kustomization.yaml`.
- `argocd-apps/` — one ArgoCD `Application` manifest per deployable component, again split into `admin/` and `apps/`, aggregated by `kustomization.yaml`. This is the **app-of-apps** root: ArgoCD is pointed at this directory, and each `Application` here in turn points `spec.source.path` back at a directory in this same repo (or the private repo) to actually deploy.
- `admin/` or `infra/` — cluster-level/system components (ArgoCD itself, cert-manager, Traefik, Cilium, hcloud-ccm, system-upgrade-controller, etc).
- `apps/` — actual workloads, one directory per app.

Each app directory (e.g. `home-k8s/apps/trilium/`, `proxima/apps/whoami/`) is a Kustomize base with its own `kustomization.yaml` listing the resources it owns — typically some subset of `deployment.yaml`, `service.yaml`, `ingress.yaml`, `pvc.yaml`, `tls.yaml`, `netpol.yaml` (home-k8s uses Kubernetes `NetworkPolicy`; proxima uses Cilium's `CiliumNetworkPolicy`, since Cilium is the CNI there). Some apps (e.g. `hass`) split into `base/` + `overlays/prod/` when a per-environment patch is needed — follow that pattern (Kustomize `patches:`) rather than duplicating manifests if you need environment-specific variants elsewhere.

## Adding a new app to a cluster

Follow the existing pattern rather than inventing a new one:

1. Create the app's manifests under `<cluster>/apps/<name>/` with a `kustomization.yaml` listing them.
2. Add a `Namespace` YAML under `<cluster>/namespaces/apps/<name>.yaml` and register it in `<cluster>/namespaces/kustomization.yaml`.
3. Add an ArgoCD `Application` under `<cluster>/argocd-apps/apps/<name>.yaml` (mirror an existing one like `trilium.yaml`/`whoami`'s — same `repoURL`, `targetRevision: HEAD`, `path: <cluster>/apps/<name>/`, `syncOptions: [ServerSideApply=true, ApplyOutOfSyncOnly=true]`) and register it in `<cluster>/argocd-apps/kustomization.yaml`.

Nothing auto-discovers new files — every new resource must be explicitly added to the relevant `kustomization.yaml` `resources:` list or ArgoCD/​Kustomize will silently ignore it.

## Security defaults (don't weaken without reason)

- Both clusters default to enforcing **UserNamespaces** and **restricted PodSecurityAdmission** profiles.
- Firewalls/NetworkPolicies are **default-deny** for both ingress and egress; new workloads need an explicit `netpol.yaml` (or `cilium-network-policy.yaml` on proxima) rather than relying on cluster defaults being open.
- On proxima, host firewall rules go through Cilium `CiliumClusterwideNetworkPolicy`, not plain iptables/security groups.
