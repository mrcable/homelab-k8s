# homelab-k8s

Een GitOps repository voor het k3s cluster op Proxmox in mijn homelab. Alle manifests worden automatisch gesynchroniseerd via ArgoCD.

## Stack

| Tool | Rol |
|------|-----|
| [k3s](https://k3s.io) | Lightweight Kubernetes (1 control plane + 3 workers) |
| [ArgoCD](https://argo-cd.readthedocs.io) | GitOps — sync van deze repo naar het cluster |
| [Metallb](https://metallb.universe.tf) | LoadBalancer IPs op bare-metal (192.168.1.210–220) |
| [Prometheus](https://prometheus.io) | Metrics scraping van nodes en pods |
| [Grafana](https://grafana.com) | Dashboards op `grafana.internal` |
| [Loki](https://grafana.com/oss/loki) | Log aggregatie — ontvangt Falco security events |
| [Falco](https://falco.org) | Runtime security — detecteert verdacht gedrag in containers via eBPF |
| [Sealed Secrets](https://github.com/bitnami-labs/sealed-secrets) | Versleutelde secrets veilig in Git |

---

## Cluster

| Node | IP | Rol |
|------|----|-----|
| k3s-00 | 192.168.1.200 | control plane |
| k3s-01 | 192.168.1.201 | worker |
| k3s-02 | 192.168.1.202 | worker |
| k3s-03 | 192.168.1.203 | worker |

Metallb IP pool: `192.168.1.210–192.168.1.220`

| Service | LoadBalancer IP | Intern domein |
|---------|----------------|---------------|
| ArgoCD | 192.168.1.211 | argocd.internal |
| Grafana | 192.168.1.212 | grafana.internal |
| Falcosidekick UI | 192.168.1.213 | falco.internal |

VMs draaien als Full Clones van een Ubuntu 24.04 Cloud Image template op Proxmox (192.168.1.5).

---

## Structuur

```
homelab-k8s/
├── apps/
│   ├── argocd/
│   │   ├── metallb-app.yaml
│   │   ├── prometheus-app.yaml
│   │   ├── grafana-app.yaml
│   │   ├── sealed-secrets-app.yaml
│   │   ├── falco-app.yaml
│   │   └── loki-app.yaml
│   ├── prometheus/
│   │   └── values.yaml
│   ├── grafana/
│   │   ├── values.yaml
│   │   └── sealed-secret.yaml
│   ├── falco/
│   │   └── values.yaml
│   └── loki/
│       └── values.yaml
├── infra/
│   └── metallb/
│       └── ipaddresspool.yaml
└── README.md
```

- `apps/argocd/` — ArgoCD Application manifests, één per tool
- `apps/<tool>/` — Helm values en overige config per applicatie
- `infra/metallb/` — IP pool config voor Metallb

---

## Secrets

Secrets worden versleuteld met [Sealed Secrets](https://github.com/bitnami-labs/sealed-secrets). Alleen de controller in het cluster kan ze ontsleutelen — versleutelde YAML is veilig om te committen.

Nieuw secret aanmaken zonder dat het in bash history terechtkomt:

```bash
read -s -p "Wachtwoord: " PASS

kubectl create secret generic <naam> \
  --from-literal=admin-user='admin' \
  --from-literal=admin-password="$PASS" \
  --namespace <namespace> \
  --dry-run=client -o yaml > /tmp/secret.yaml

unset PASS

kubeseal --controller-name=sealed-secrets \
  --controller-namespace=kube-system \
  --format yaml < /tmp/secret.yaml > apps/<tool>/sealed-secret.yaml

rm /tmp/secret.yaml
```

---

## Gerelateerde repo

**[mrcable/homelab](https://github.com/mrcable/homelab)** — Docker stack op Ubuntu Server (Pi-hole, Caddy, Unifi Controller)
