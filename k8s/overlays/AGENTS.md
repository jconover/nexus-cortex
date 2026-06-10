<!-- Parent: ../AGENTS.md -->
<!-- Generated: 2026-06-10 | Updated: 2026-06-10 -->

# overlays

## Purpose
Kustomize overlays that layer environment-specific configuration on top of the base manifests
in `k8s/`. Each overlay is a `kustomization.yaml` that patches the base for its environment.

## Subdirectories

| Directory | Purpose |
|-----------|---------|
| `dev/` | Development overrides (`kustomization.yaml`) |
| `staging/` | Staging overrides (`kustomization.yaml`) |
| `production/` | Production overrides (`kustomization.yaml`) |

## For AI Agents

### Working In This Directory
- Put environment-specific changes (replica counts, image tags, resource limits, ingress hosts)
  in the appropriate overlay — never in the base manifests.
- Render before applying: `kubectl kustomize k8s/overlays/<env>` (or
  `kubectl apply -k k8s/overlays/<env>`).

## Dependencies

### Internal
- The base resources in `k8s/` referenced via the overlays' `resources:`/`bases:` entries.

### External
- Kustomize / kubectl.

<!-- MANUAL: -->
