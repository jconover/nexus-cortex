<!-- Parent: ../AGENTS.md -->
<!-- Generated: 2026-06-10 | Updated: 2026-06-10 -->

# app

## Purpose
The heart of the backend. Contains the FastAPI application, the RAG pipeline, all external
service integrations (Ollama, Qdrant, Redis, PostgreSQL), and the supporting production
subsystems: analytics, authentication, A/B testing, drift detection, circuit breakers,
semantic caching, conversation memory, and observability. A newer layered `retrieval/` package
provides a clean strategy/expander/generator abstraction over the original monolithic `rag.py`.

## Key Files

### API & Orchestration
| File | Description |
|------|-------------|
| `main.py` | FastAPI app + all endpoints (`/api/chat`, `/chat/stream`, health/live/ready, models, stats, analytics, experiments, auth, circuit-breakers, drift, freshness, GPU/Ollama status); middleware (request-ID, analytics), rate limiting, lifespan startup/shutdown |
| `rag.py` | Original RAG pipeline: `generate_response()`, `generate_response_stream()`, score-aware retrieval, reranking, metrics |
| `agent.py` | Agentic RAG loop (PLAN→RETRIEVE→REFLECT→DECIDE→GENERATE→VERIFY) built on `rag.py` |
| `config.py` | Centralized settings (Pydantic) read from `.env`; validation |
| `models.py` | Pydantic request/response models for all endpoints |
| `templates.py` | Prompt template catalog + render/validate helpers |

### Retrieval & Generation
| File | Description |
|------|-------------|
| `vectorstore.py` | Qdrant interface: hybrid (BM25+vector) search, embedding cache, stats (`VectorStore`) |
| `reranker.py` | Cross-encoder reranking of candidate documents |
| `sparse_encoder.py` | BM25 sparse vector encoding for hybrid search |
| `query_expansion.py` | HyDE (Hypothetical Document Embeddings) query expansion |
| `conversation_context.py` | Conversation-aware query expansion for follow-up questions |
| `context_compression.py` | Contextual compression of retrieved documents |
| `web_search.py` | Tavily web-search fallback (circuit-breaker protected) |
| `semantic_cache.py` | Semantic response cache for similar queries |
| `llm_provider.py` | Multi-provider LLM abstraction layer |

### Persistence & Memory
| File | Description |
|------|-------------|
| `database.py` | Async SQLAlchemy engine/session management for PostgreSQL |
| `db_models.py` | SQLAlchemy ORM models (QueryLog, Feedback, Experiment*, User, APIKey, …) |
| `redis_client.py` | Shared Redis connection pool (conversation memory + embedding cache) |
| `conversation_storage.py` | Tiered conversation storage with automatic summarization |

### Production Subsystems
| File | Description |
|------|-------------|
| `auth.py` | Authentication service (password hashing, JWT, API keys, user dependencies) |
| `analytics.py` | Real-time operational metrics collector |
| `metrics.py` | Retrieval metrics logging + Prometheus instrumentation |
| `ab_testing.py` | A/B testing service for pipeline experimentation |
| `drift_detection.py` | Embedding/retrieval-quality drift detection |
| `circuit_breaker.py` | Circuit breakers for Ollama/Qdrant/Tavily resilience |
| `evaluation.py` / `rag_eval.py` | RAG quality evaluation (incl. RAGAS scoring) |
| `output_validation.py` | Output validation / hallucination detection |
| `feedback.py` | User feedback logging |
| `device_utils.py` | GPU/CPU/MPS device detection for ML models |
| `tracing.py` | OpenTelemetry distributed tracing |

## Subdirectories

| Directory | Purpose |
|-----------|---------|
| `retrieval/` | Layered RAG abstraction: pipeline orchestrator, strategies, expanders, generators (see `retrieval/AGENTS.md`) |
| `repositories/` | Repository pattern for data access (see `repositories/AGENTS.md`) |

## For AI Agents

### Working In This Directory
- **Two RAG paths coexist:** the original `rag.py` pipeline (used by `/api/chat`) and the newer
  `retrieval/` package (orchestrator + strategy/expander/generator). `agent.py` reuses `rag.py`
  internals. When changing retrieval behavior, check which path the endpoint actually calls.
- **Singletons:** `rag_pipeline`, `vector_store`, `conversation_storage`, `drift_detector`,
  `auth_service`, etc. are module-level instances. Imports inside methods are intentional to
  break circular dependencies and defer heavy ML loads — preserve that pattern.
- **Feature flags:** Almost every subsystem is toggled via `config.py`/`.env`
  (`HYBRID_SEARCH_ENABLED`, `RERANKER_ENABLED`, `HYDE_ENABLED`, `WEB_SEARCH_ENABLED`,
  `SEMANTIC_CACHE_ENABLED`, `ANALYTICS_ENABLED`, `QUERY_LOGGING_ENABLED`,
  `CONVERSATION_SUMMARIZATION_ENABLED`, etc.). Always degrade gracefully when a flag is off or a
  dependency (PostgreSQL, Tavily) is unavailable.
- **PostgreSQL is optional:** the app starts and serves chat even if `init_db()` fails; query
  logging and analytics persistence are fire-and-forget background tasks that must never break
  the request flow.
- Adding an endpoint: define Pydantic models in `models.py`, add the route in `main.py`, reuse
  the singletons.

### Testing Requirements
- `cd backend && pytest`. See `tests/` for API/RAG/validation/cache/template coverage.

### Common Patterns
- Async/await for all I/O; `HTTPException` for API errors; structured logging via `logging`.
- Background work uses the `_create_background_task` helper in `main.py` (keeps a reference,
  logs unhandled exceptions, auto-cleans on completion).
- Result objects carry rich metadata (`retrieval_metrics`, timings) for analytics/observability.

## Dependencies

### Internal
- `retrieval/`, `repositories/`, and (dynamically) `scripts/freshness_tracker.py`.

### External
- FastAPI, Pydantic, SQLAlchemy (async), slowapi, ollama, qdrant-client, redis,
  sentence-transformers/torch, langchain, OpenTelemetry, RAGAS, Tavily.

<!-- MANUAL: -->
