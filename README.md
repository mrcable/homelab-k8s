# homelab-k8s

GitOps repository voor het k3s cluster op Proxmox.
Beheerd via ArgoCD, draait in het cluster.

## Structuur

- `apps/` — applicaties (ArgoCD, Grafana, Prometheus)
- `infra/` — infrastructuur (Metallb)
