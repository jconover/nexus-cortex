<!-- Parent: ../AGENTS.md -->
<!-- Generated: 2026-06-10 | Updated: 2026-06-10 -->

# backend

## Purpose
The FastAPI backend that powers the DevOps AI Assistant. Hosts the RAG pipeline, all API
endpoints, vector/LLM/Redis/PostgreSQL integrations, and the supporting subsystems
(analytics, auth, A/B testing, drift detection, circuit breakers, tracing). Containerized
for both CPU and GPU inference.

## Key Files

| File | Description |
|------|-------------|
| `Dockerfile` | CPU image build |
| `Dockerfile.gpu` | GPU (CUDA cu121) image build for GPU embeddings/reranking |
| `requirements.txt` | Python dependencies (CPU) |
| `requirements-gpu.txt` | Python dependencies (GPU / torch+cu121) |
| `pytest.ini` | Pytest configuration |

## Subdirectories

| Directory | Purpose |
|-----------|---------|
| `app/` | Application code: RAG pipeline, API, services (see `app/AGENTS.md`) |
| `tests/` | Pytest suite for API and RAG components (see `tests/AGENTS.md`) |

## For AI Agents

### Working In This Directory
- The app is imported as the `app` package (e.g. `from app.config import settings`), so run
  commands from the `backend/` directory or with `backend/` on `PYTHONPATH`.
- CPU vs GPU is selected by which Dockerfile/requirements file is used; both share the same
  source. Don't hardcode device — use `app/device_utils.py` helpers.

### Testing Requirements
- Run `pytest` from `backend/`. Tests use fixtures in `tests/conftest.py`.

### Common Patterns
- Async I/O everywhere; centralized settings in `app/config.py`.
- Heavy/shared objects are lazy-loaded singletons to avoid circular imports and startup cost.

## Dependencies

### Internal
- `scripts/` — ingestion and freshness tracking (imported dynamically by some endpoints)

### External
- FastAPI, Pydantic, SQLAlchemy (async), slowapi (rate limiting), ollama, qdrant-client,
  redis, sentence-transformers / torch, langchain, OpenTelemetry, RAGAS (eval)

<!-- MANUAL: -->
