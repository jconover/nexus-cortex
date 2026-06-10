<!-- Generated: 2026-06-10 | Updated: 2026-06-10 -->

# nexus-cortex (DevOps AI Assistant)

## Purpose
A production-oriented RAG (Retrieval-Augmented Generation) system that answers DevOps
documentation questions using local LLMs via Ollama. The system indexes 30+ documentation
sources into Qdrant and serves answers through a FastAPI backend with hybrid search,
cross-encoder reranking, HyDE query expansion, conversation memory, web-search fallback,
and an optional agentic RAG loop. A React frontend provides the chat UI. The platform also
includes PostgreSQL-backed analytics, authentication, A/B testing, drift detection, and
Kubernetes deployment manifests.

**Tech Stack:** FastAPI (Python) + React + Ollama (LLM) + Qdrant (vector DB) +
Redis (conversation memory / embedding cache) + PostgreSQL (analytics, auth, experiments).

## Key Files

| File | Description |
|------|-------------|
| `Makefile` | Primary entrypoint for all operations (start/stop, ingest, health, aider) |
| `docker-compose.yml` | Production stack (Docker Hub images) |
| `docker-compose.dev.yml` | Development stack (local builds, hot-reload) |
| `.env` / `.env.example` | Environment configuration (services + advanced RAG feature flags) |
| `CLAUDE.md` | Primary guidance file for AI agents (commands, architecture, conventions) |
| `ARCHITECTURE.md` | Detailed architecture documentation |
| `README.md` | Project overview and getting started |
| `ROADMAP.md` / `IMPROVEMENT_RECOMMENDATIONS.md` | Forward-looking planning docs |
| `get-docker.sh` | Docker install helper |

> This repo root contains many top-level Markdown guides (SETUP, QUICKSTART, RESTART_GUIDE,
> PROJECT_GUIDE, etc.). They are human documentation; `CLAUDE.md` is the canonical agent guide.

## Subdirectories

| Directory | Purpose |
|-----------|---------|
| `backend/` | FastAPI backend, RAG pipeline, and tests (see `backend/AGENTS.md`) |
| `frontend/` | React chat UI (see `frontend/AGENTS.md`) |
| `scripts/` | Ingestion, benchmarking, ops, and migration scripts (see `scripts/AGENTS.md`) |
| `k8s/` | Kubernetes manifests with Kustomize overlays (see `k8s/AGENTS.md`) |
| `n8n-workflows/` | n8n automation workflows for scheduled doc updates (see `n8n-workflows/AGENTS.md`) |
| `docs/` | Supplementary roadmap and learning docs (see `docs/AGENTS.md`) |
| `data/` | Documentation sources, custom uploads, and eval datasets (see `data/AGENTS.md`) |
| `.github/` | CI/CD workflows (ci, docker-publish, security) |
| `.claude/` | Repo-local Claude Code subagent definitions |

## For AI Agents

### Working In This Directory
- **Never run services directly** — use the `Makefile`. `make start-dev` for local builds with
  hot-reload, `make start` for Docker Hub images, `make stop` to tear down.
- The `.env` file drives nearly all behavior. Changing `EMBEDDING_MODEL`, `EMBEDDING_DIMENSION`,
  or `HYBRID_SEARCH_ENABLED` requires full re-ingestion: `make ingest-recreate`.
- Backend code changes need `docker compose restart backend` (or rely on dev hot-reload).
- `CLAUDE.md` describes the system as embedding-on-CPU, but recent commits run embeddings and
  the reranker on GPU (cu121). Verify device via `GET /api/health` (`device_info`) rather than
  assuming.

### Testing Requirements
- Backend tests: `cd backend && pytest` (config in `backend/pytest.ini`).
- Service health after changes: `make health`; vector DB populated: `make stats`.

### Common Patterns
- Async/await for all I/O; type hints throughout; `HTTPException` for API errors.
- Shared resources are module-level singletons (`rag_pipeline`, `vector_store`, etc.).
- Configuration is centralized in `backend/app/config.py` and read from environment.

## Dependencies

### External
- Ollama (local LLM inference, GPU), Qdrant (vector DB), Redis, PostgreSQL
- Tavily API (optional web-search fallback)
- Docker / Docker Compose / Kubernetes for deployment

<!-- MANUAL: Add project-specific notes below this line; preserved on regeneration. -->
