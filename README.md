# Kawaz K8s

Kubernetes manifests for the [Kawaz](https://kawazplus.com) video streaming platform, deployed on GKE.

## Services

| Service | Image | Description |
|---|---|---|
| `kawaz-frontend` | `ghcr.io/kawaz-video-streaming/kawaz-frontend` | Frontend (Next.js) |
| `kawaz-backend` | `ghcr.io/kawaz-video-streaming/kawaz-backend` | Backend API (port 8080) |
| `media-processor` | `ghcr.io/kawaz-video-streaming/media-processor` | GPU-accelerated video transcoding (port 8081) |

## Architecture notes

- All resources live in the `kawaz` namespace.
- `media-processor` runs on a dedicated GPU node pool (`gpu-pool`) using GKE Spot instances to reduce cost.
- `media-processor` is scaled to 0 replicas by default and scaled up automatically via **KEDA** when messages appear in the `media-processor-convert` RabbitMQ queue.
- Images are pulled from GHCR and require the `ghcr-secret` image pull secret.

## Prerequisites

- GKE cluster with a GPU node pool (`gpu-pool`) that tolerates `nvidia.com/gpu` and `cloud.google.com/gke-spot`
- [KEDA](https://keda.sh) installed in the cluster
- NGINX Ingress Controller installed
- The following Kubernetes Secrets created manually in the `kawaz` namespace:

| Secret name | Keys |
|---|---|
| `ghcr-secret` | Docker registry credentials for `ghcr.io` |
| `kawaz-backend-secrets` | App secrets (DB connection, JWT, etc.) |
| `media-processor-secrets` | `RABBITMQ_USER`, `RABBITMQ_PASS`, and any other processor secrets |

- The following ConfigMaps created manually:

| ConfigMap name | Used by |
|---|---|
| `kawaz-backend-config` | `kawaz-backend` |
| `media-processor-config` | `media-processor` |

## Deploying

```bash
# 1. Create namespace
kubectl apply -f namespace.yaml

# 2. Create secrets and configmaps manually (not tracked in git)

# 3. Apply all manifests
kubectl apply -f kawaz-backend/
kubectl apply -f kawaz-frontend/
kubectl apply -f media-processor/
kubectl apply -f ingress.yaml
```

## Ingress

Update `ingress.yaml` with your actual domain before applying — the `host` field is currently set to `yourdomain.com`.
