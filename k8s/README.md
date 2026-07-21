# AI Task Platform — Kubernetes Infrastructure

## Directory Structure

```
k8s/
├── base/                  # Shared base configuration
│   ├── namespace.yaml     # ai-task-platform namespace
│   ├── configmap.yaml     # Shared environment variables
│   ├── secrets.example.yaml  # Secret template (do not commit real secrets)
│   ├── ingress.yaml       # nginx ingress routing / and /api
│   ├── mongodb/           # MongoDB Stateful workload
│   ├── redis/             # Redis cache
│   ├── api/               # Express API backend
│   ├── worker/            # Python worker
│   └── web/               # Next.js frontend
├── overlays/
│   ├── staging/           # Staging: 1 replica, smaller resources
│   └── production/        # Production: full replica counts, larger resources
└── README.md
```

## Prerequisites

- Kubernetes cluster (k3s-compatible)
- nginx ingress controller
- Argo CD (optional, for GitOps)

## Usage

### Apply staging directly

```bash
kubectl apply -k k8s/overlays/staging
```

### Apply production directly

```bash
kubectl apply -k k8s/overlays/production
```

### Preview rendered manifests

```bash
kubectl kustomize k8s/overlays/production
```

## Secrets

The `secrets.example.yaml` file shows the required secret keys. Do not commit real secrets.

For production, override secrets using:
- `kubectl create secret generic ai-task-platform-secrets --namespace ai-task-platform --from-literal=JWT_SECRET=...`
- Or update the `secretGenerator` literals in the production overlay.

## Resource Allocation

| Service  | Staging Requests     | Production Requests  | Staging Limits       | Production Limits    |
|----------|---------------------|----------------------|----------------------|----------------------|
| MongoDB  | —                   | 256Mi / 500m         | —                    | 1Gi / 1000m          |
| Redis    | —                   | 64Mi / 100m          | —                    | 256Mi / 500m         |
| API      | 64Mi / 100m         | 256Mi / 400m         | 256Mi / 250m         | 1Gi / 1000m          |
| Worker   | 64Mi / 100m         | 256Mi / 400m         | 256Mi / 250m         | 1Gi / 1000m          |
| Web      | 64Mi / 100m         | 256Mi / 400m         | 256Mi / 250m         | 1Gi / 1000m          |

## Replica Counts

| Service  | Staging | Production |
|----------|---------|------------|
| MongoDB  | 1       | 1          |
| Redis    | 1       | 1          |
| API      | 1       | 2          |
| Worker   | 1       | 3          |
| Web      | 1       | 2          |

## Ingress Routes

| Path   | Backend Service | Port |
|--------|----------------|------|
| `/api` | api            | 4000 |
| `/`    | web            | 3000 |

Uses nginx ingress controller. Install with:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml
```

## Argo CD

The Argo CD application manifest is at `../argocd/application.yaml`. It points to `k8s/overlays/production` and enables automated sync, self-heal, and prune.
