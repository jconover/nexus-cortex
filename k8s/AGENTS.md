<!-- Parent: ../AGENTS.md -->
<!-- Generated: 2026-06-10 | Updated: 2026-06-10 -->

# k8s

## Purpose
Production-ready Kubernetes manifests for deploying the full stack (backend, frontend,
Qdrant, Redis, PostgreSQL, Ollama) using Kustomize with per-environment overlays. See the
existing `README.md` here for the detailed deployment runbook; this file is the agent-facing
summary.

## Key Files

| File | Description |
|------|-------------|
| `README.md` | Full deployment runbook (prerequisites, secrets, quick start) |
| `kustomization.yaml` | Base Kustomize configuration (references all base manifests) |
| `namespace.yaml` | Dedicated namespace |
| `configmap.yaml` | Non-sensitive configuration |
| `secret.yaml` | Secret template — replace placeholders before applying |
| `backend-deployment.yaml` | Backend API deployment with liveness/readiness probes |
| `backend-service.yaml` | ClusterIP service for the backend |
| `backend-hpa.yaml` | HorizontalPodAutoscaler (2–10 replicas) |
| `backend-pdb.yaml` | PodDisruptionBudget for the backend |
| `frontend-deployment.yaml` | React frontend deployment |
| `qdrant-statefulset.yaml` | Vector DB with persistent storage |
| `postgres-statefulset.yaml` | PostgreSQL with persistent storage |
| `redis-deployment.yaml` | Redis cache with persistence |
| `ollama-deployment.yaml` | LLM inference (GPU node scheduling) |
| `ingress.yaml` | Ingress with TLS placeholder |
| `networkpolicy.yaml` | Network isolation policies |

## Subdirectories

| Directory | Purpose |
|-----------|---------|
| `overlays/` | Kustomize overlays for dev/staging/production (see `overlays/AGENTS.md`) |

## For AI Agents

### Working In This Directory
- Base manifests live here; environment-specific changes belong in `overlays/<env>/`, not in
  the base files.
- `secret.yaml` is a template with placeholders — never commit real secrets. Probes in
  `backend-deployment.yaml` map to `/api/health/live` and `/api/health/ready`.
- Ollama requires GPU nodes (NVIDIA device plugin); the readiness probe gates traffic until
  the embedding model, Ollama, Qdrant, and Redis are all ready.

### Testing Requirements
- Validate with `kubectl kustomize k8s/overlays/<env>` before applying.

## Dependencies

### External
- Kubernetes 1.25+, kubectl, Kustomize, NGINX Ingress Controller, optional cert-manager,
  optional NVIDIA GPU device plugin

<!-- MANUAL: -->
