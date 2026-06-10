<!-- Parent: ../AGENTS.md -->
<!-- Generated: 2026-06-10 | Updated: 2026-06-10 -->

# scripts

## Purpose
Operational and data-pipeline scripts: documentation download/ingestion, RAG benchmarking
and model comparison, chunking utilities, freshness tracking, database backup/partitioning,
Docker publishing, and the Aider AI-coding setup. These are invoked primarily through the
root `Makefile` targets (e.g. `make ingest`, `make download-docs`, `make update-docs`).

## Key Files

| File | Description |
|------|-------------|
| `ingest_docs.py` | Document ingestion pipeline: load → split → embed → upsert into Qdrant |
| `download_docs.sh` | Git-clones the documentation source repositories into `data/docs/` |
| `update_docs.sh` | Updates cloned docs to latest versions |
| `chunkers.py` | Text chunking strategies used during ingestion |
| `chunk_deduplication.py` | Deduplicates near-identical chunks before indexing |
| `ingestion_registry.py` | Tracks what has been ingested (source registry) |
| `freshness_tracker.py` | Tracks doc source staleness; backs `GET /api/docs/freshness` |
| `benchmark_rag.py` | Benchmarks RAG retrieval/answer quality |
| `compare_models.py` | Compares LLM models on the same queries |
| `create_partitions.py` | Creates PostgreSQL partitions for query logs |
| `backup_databases.sh` | Backup helper for Qdrant/Redis/PostgreSQL |
| `push_to_dockerhub.sh` | Builds and pushes images to Docker Hub |
| `setup_aider.sh` / `test_aider.sh` | Aider AI-coding tool install + smoke test |
| `verify_setup.sh` / `test_api.sh` | Environment verification and API smoke tests |

## Subdirectories

| Directory | Purpose |
|-----------|---------|
| `migrations/` | SQL migrations (e.g. query-log partitioning) (see `migrations/AGENTS.md`) |

## For AI Agents

### Working In This Directory
- Prefer running these via `Makefile` targets so paths and env vars are set consistently.
- `ingest_docs.py` must stay consistent with the backend embedding config — re-ingest with
  `make ingest-recreate` whenever embedding model/dimension or hybrid-search settings change.
- Adding a documentation source: edit `download_docs.sh` (clone) AND the `doc_sources` dict in
  `ingest_docs.py`, then `make download-docs && make ingest`.
- Some scripts are imported at runtime by the backend (e.g. `freshness_tracker`); keep their
  module-level import side effects light.

### Testing Requirements
- `verify_setup.sh` and `test_api.sh` provide quick smoke checks against a running stack.

## Dependencies

### Internal
- Shares embedding/vectorstore assumptions with `backend/app/vectorstore.py` and `config.py`.

### External
- LangChain loaders/splitters, HuggingFace embeddings, qdrant-client, git, Docker CLI

<!-- MANUAL: -->
