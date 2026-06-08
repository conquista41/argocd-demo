# argocd-demo

A GitOps demo using [ArgoCD](https://argo-cd.readthedocs.io/) and [Kustomize](https://kustomize.io/) to manage a simple nginx application across `dev` and `prod` environments.

## Repository Structure

```
argocd-demo/
└── apps/
    └── my-app/
        ├── base/                  # Shared base manifests
        │   ├── deployment.yaml
        │   ├── service.yaml
        │   └── kustomization.yaml
        └── overlays/
            ├── dev/               # Dev environment overrides
            │   └── kustomization.yaml
            └── prod/              # Prod environment overrides
                └── kustomization.yaml
```

## Application

| Property | Value |
|---|---|
| Image | `nginx:1.27` |
| Port | `80` |
| Service type | `ClusterIP` |

## Environments

| Environment | Namespace | Replicas |
|---|---|---|
| `dev` | `dev` | 1 |
| `prod` | `prod` | 3 |

## Prerequisites

- A running Kubernetes cluster
- [kubectl](https://kubernetes.io/docs/tasks/tools/) configured for your cluster
- [ArgoCD](https://argo-cd.readthedocs.io/en/stable/getting_started/) installed in the cluster
- [kustomize](https://kubectl.docs.kubernetes.io/installation/kustomize/) (or `kubectl` v1.14+)

## Usage

### Preview rendered manifests locally

```bash
# Dev overlay
kubectl kustomize apps/my-app/overlays/dev

# Prod overlay
kubectl kustomize apps/my-app/overlays/prod
```

### Apply directly with kubectl

```bash
kubectl apply -k apps/my-app/overlays/dev
kubectl apply -k apps/my-app/overlays/prod
```

### Register with ArgoCD

```bash
argocd app create my-app-dev \
  --repo https://github.com/<your-org>/argocd-demo \
  --path apps/my-app/overlays/dev \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace dev \
  --sync-policy automated

argocd app create my-app-prod \
  --repo https://github.com/<your-org>/argocd-demo \
  --path apps/my-app/overlays/prod \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace prod \
  --sync-policy automated
```

## How It Works

The project follows the **base + overlays** Kustomize pattern:

- **`base/`** defines the core Deployment and Service resources shared by all environments.
- **`overlays/dev/`** and **`overlays/prod/`** each reference the base and apply JSON patches to set environment-specific values (namespace, replica count).
- ArgoCD watches this repository and automatically syncs each overlay to its target namespace whenever changes are pushed to `main`.
