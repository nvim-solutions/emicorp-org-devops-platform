# emicorp-org-devops-platform

Platform / GitOps configuration repository for the Emicorp Platform Engineering Lab.

This repository is the **source of truth for cluster state**: cluster definitions, internal infrastructure charts (databases, monitoring, etc.), per-environment value overrides, and (eventually) ArgoCD `Application` manifests.

## Layout

```
.
├── kind/
│   └── cluster-config.yaml           ← Kind multi-node cluster definition
│
├── charts/                           ← Internal, repo-local Helm charts
│   └── postgres/                     ← Minimal Postgres chart (incl. shared secret)
│       ├── Chart.yaml
│       ├── values.yaml
│       └── templates/
│           ├── deployment.yaml
│           ├── service.yaml
│           └── secret.yaml
│
├── environments/                     ← Per-environment value overrides
│   ├── dev/
│   │   ├── postgres.values.yaml
│   │   └── users-service.values.yaml
│   └── prod/
│       ├── postgres.values.yaml
│       └── users-service.values.yaml
│
└── argocd/                           ← (Phase 4) ArgoCD Application manifests
```

## Companion repositories

| Concern | Repository |
|---|---|
| Application code | `emicorp-org-devops-spring-users-service` |
| Application Helm chart | `emicorp-org-devops-helm-users-service` |
| Platform / GitOps (this repo) | `emicorp-org-devops-platform` |

---

## Local cluster lifecycle

```bash
kind create cluster --config kind/cluster-config.yaml
kind get clusters
kind delete cluster --name emicorp-org
```

## Deploy infra (Postgres + shared secret)

```bash
helm upgrade --install postgres ./charts/postgres \
  -f environments/dev/postgres.values.yaml
```

## Deploy the application

Assumes the `emicorp-org-devops-helm-users-service` repo is cloned as a sibling directory:

```bash
helm upgrade --install users \
  ../emicorp-org-devops-helm-users-service \
  -f environments/dev/users-service.values.yaml
```

To switch environment, change the `-f` file. Same chart, different values.

## Conventions

- **One values file per (service, environment)**, named `<service>.values.yaml`.
- **Only deltas** live in environment files; chart defaults stay in the chart's `values.yaml`.
- Production values **never** contain real secrets — wired through external secret stores later (Phase 6).
