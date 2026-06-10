# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**DevOps AI Assistant** - A RAG (Retrieval-Augmented Generation) system using local LLMs via Ollama to answer questions about DevOps documentation. 30+ documentation sources indexed in Qdrant vector database.

Beyond the core RAG loop, the system has grown into a production platform: PostgreSQL-backed
analytics and query logging, optional JWT/API-key authentication, A/B testing, drift detection,
circuit breakers, OpenTelemetry tracing, a multi-provider LLM layer (Ollama or Anthropic), an
agentic RAG loop, and Kubernetes deployment manifests. A layered `retrieval/` package provides a
clean strategy/expander/generator abstraction alongside the original `rag.py` pipeline.

**Tech Stack**: FastAPI (Python) + React + Ollama (LLM) + Qdrant (vector DB) + Redis (conversation memory & embedding cache) + PostgreSQL (analytics, auth, A/B experiments)

## Common Commands

```bash
# Start/Stop
make start              # Production (Docker Hub images)
make start-dev          # Development (local builds + hot-reload)
make stop               # Stop all services
make logs-backend       # View backend logs

# First-time setup
make pull-model         # Pull llama3.1:8b (default)
make pull-model MODEL=mistral:7b  # Pull specific model
make ingest             # Download and index all documentation
make ingest-recreate    # Ingest and recreate collection on schema mismatch

# Operations
make health             # Service health check
make stats              # Vector DB statistics
make update-docs        # Update docs to latest versions

# AI Coding (Aider)
make setup-aider        # Install Aider + Qwen2.5-Coder
make aider              # Start with qwen2.5-coder:7b
make aider-32b          # Start with qwen2.5-coder:32b
```

### Testing API Endpoints
```bash
curl http://localhost:8000/api/health
curl -X POST http://localhost:8000/api/chat \
  -H "Content-Type: application/json" \
  -d '{"message": "How do I create a K8s deployment?", "model": "llama3.1:8b"}'

# Streaming response
curl --no-buffer -X POST http://localhost:8000/api/chat/stream \
  -H "Content-Type: application/json" \
  -d '{"message": "Explain Kubernetes pods", "model": "llama3.1:8b"}'
```

## Architecture

```
Frontend (React:3000) → Backend (FastAPI:8000) → Ollama   (LLM:11434)
                              ↓                    Qdrant   (Vector:6333)
                              ↓                    Redis    (Memory/cache:6379)
                              ↓                    Postgres (Analytics/auth:5432)
```

### RAG Pipeline Flow
1. User query → Backend `/api/chat` or `/api/chat/stream`
2. **Semantic cache** (optional): Return a cached response for a semantically similar prior query
3. **Conversation context** (optional): Resolve pronouns ("it", "that") using prior messages
4. **HyDE expansion** (optional): Generate hypothetical document for vague queries
5. **Hybrid search**: BM25 keyword + vector semantic search with RRF fusion
6. **Reranking**: Cross-encoder reranks top 20 → top 5
7. **Web search fallback**: If scores low, Tavily searches trusted doc sites
8. Context + query → Ollama LLM → Response with source attribution
9. **Output validation** (optional): Hallucination/grounding check on the response
10. Conversation saved to Redis (24h TTL); query logged to PostgreSQL (fire-and-forget)

> **Two RAG paths coexist.** `/api/chat` and `/api/chat/stream` use the original `rag.py`
> pipeline. The newer `backend/app/retrieval/` package re-implements the same flow behind
> composable `RetrievalStrategy` / `QueryExpander` / `ResponseGenerator` interfaces wired by
> `RAGPipelineOrchestrator`. The **agentic** path (`/api/agent/chat`, `agent.py`) wraps `rag.py`
> in a PLAN→RETRIEVE→REFLECT→DECIDE→GENERATE→VERIFY loop. When changing retrieval behavior,
> confirm which path the target endpoint actually calls.

### Key Files

**Core RAG & API**

| File | Purpose |
|------|---------|
| `backend/app/rag.py` | Original RAG pipeline: `generate_response()`, `generate_response_stream()`, `_build_messages()` |
| `backend/app/main.py` | FastAPI app + all endpoints, middleware, rate limiting, lifespan startup/shutdown |
| `backend/app/agent.py` | Agentic RAG loop (`/api/agent/chat`): plan → retrieve → reflect → decide → generate → verify |
| `backend/app/vectorstore.py` | Qdrant interface, hybrid search, embedding cache (`VectorStore` class) |
| `backend/app/reranker.py` | Cross-encoder reranking |
| `backend/app/query_expansion.py` | HyDE query expansion |
| `backend/app/conversation_context.py` | Conversation-aware query expansion for follow-ups |
| `backend/app/conversation_storage.py` | Tiered conversation storage + automatic summarization |
| `backend/app/web_search.py` | Tavily web search fallback (circuit-breaker protected) |
| `backend/app/semantic_cache.py` | Semantic response caching for similar queries |
| `backend/app/llm_provider.py` | Multi-provider LLM abstraction (Ollama / Anthropic) |
| `backend/app/output_validation.py` | Output validation / hallucination detection |
| `backend/app/config.py` | Environment configuration with validation (reads `.env`) |

**Layered retrieval package** (`backend/app/retrieval/`)

| File | Purpose |
|------|---------|
| `retrieval/pipeline.py` | `RAGPipelineOrchestrator` — expand → retrieve → generate |
| `retrieval/strategies/hybrid.py` | `HybridRetrievalStrategy` (vector+BM25, rerank, web fallback) |
| `retrieval/expanders/` | `ConversationExpander`, `HyDEExpander`, `CompositeExpander` |
| `retrieval/generators/ollama.py` | `OllamaGenerator` (sync + streaming) |

**Persistence & production subsystems**

| File | Purpose |
|------|---------|
| `backend/app/database.py` | Async SQLAlchemy engine/session management (PostgreSQL) |
| `backend/app/db_models.py` | ORM models: QueryLog, Feedback, Experiment*, User, APIKey |
| `backend/app/repositories/` | Repository-pattern data access (base, conversation, query_log, vector) |
| `backend/app/auth.py` | Authentication: password hashing, JWT, API keys, user dependencies |
| `backend/app/analytics.py` | Real-time operational metrics collector |
| `backend/app/metrics.py` | Retrieval metrics + Prometheus instrumentation |
| `backend/app/ab_testing.py` | A/B testing service for pipeline experimentation |
| `backend/app/drift_detection.py` | Embedding/retrieval-quality drift detection |
| `backend/app/circuit_breaker.py` | Circuit breakers for Ollama/Qdrant/Tavily |
| `backend/app/tracing.py` | OpenTelemetry distributed tracing |
| `backend/app/device_utils.py` | GPU/CPU/MPS device detection for ML models |

**Ingestion & ops**

| File | Purpose |
|------|---------|
| `scripts/ingest_docs.py` | Document ingestion pipeline |
| `scripts/download_docs.sh` | Git clone documentation repos |
| `scripts/freshness_tracker.py` | Doc source staleness tracking (backs `/api/docs/freshness`) |
| `k8s/` | Kubernetes manifests with Kustomize overlays (dev/staging/production) |

### API Surface (`backend/app/main.py`)

| Group | Endpoints |
|-------|-----------|
| Chat | `POST /api/chat`, `POST /api/chat/stream`, `POST /api/agent/chat` |
| Health | `GET /api/health`, `/api/health/live`, `/api/health/ready` |
| Models/Stats | `GET /api/models`, `/api/stats`, `/api/ollama-status`, `/api/gpu-metrics` |
| Conversation | `DELETE /api/conversation/{id}`, `GET /api/conversation/{id}/context`, `GET /api/conversation/{id}/stats`, `GET /api/history/{id}` |
| Templates | `GET /api/templates` (+ `/{id}`), `POST /api/templates/render` |
| Upload | `POST /api/upload` |
| Feedback | `POST /api/feedback`, `GET /api/feedback/summary` |
| Auth | `POST /api/auth/register`, `/login`, `/logout`; `GET/PUT /api/auth/me`; `*/api/auth/api-keys` |
| A/B Testing | `*/api/experiments` (+ `/{id}`, `/assignment`) |
| Analytics | `GET /api/analytics/realtime`, `/api/analytics/queries` (+ `/summary`), `/api/metrics/retrieval` |
| Drift | `GET /api/metrics/drift` (+ `/status`, `/history`), `POST /api/metrics/drift/baseline`, `/reset` |
| Resilience | `GET /api/circuit-breakers`, `POST /api/circuit-breakers/reset` |
| Eval / Docs | `POST /api/eval/run`, `GET /api/docs/freshness` |

> Interactive OpenAPI docs are served at `/docs`.

### Document Ingestion Flow
```
Markdown/Text → LangChain DirectoryLoader → RecursiveCharacterTextSplitter
(chunk_size=1000, overlap=200) → HuggingFace Embeddings → Qdrant
```

## Implementation Details

### RAG Configuration
- **Chunk Size**: 1000 chars with 200 overlap
- **Embedding**: `BAAI/bge-base-en-v1.5` (768 dims). Device is `auto` by default — uses CUDA/MPS
  when available, else CPU. The reranker is also `auto`. The GPU images (`Dockerfile.gpu`,
  `requirements-gpu.txt`, cu121) run embeddings and reranking on GPU. Verify the actual device
  via `GET /api/health` (`device_info`).
- **Top K**: 5 documents per query (initial retrieval over-fetches `RETRIEVAL_TOP_K=20` when the reranker is enabled)
- **Collection**: `devops_docs` in Qdrant

> **Note**: Changing the embedding model or toggling `HYBRID_SEARCH_ENABLED` requires full re-ingestion of all documents. Run `make ingest-recreate` after modifying `EMBEDDING_MODEL`, `EMBEDDING_DIMENSION`, or `HYBRID_SEARCH_ENABLED` in `.env`.

### Environment Variables (`.env`)

> All settings live in `backend/app/config.py` (`Settings`, pydantic) with validation of
> interdependent values. The `=value` shown below is the **code default** in `config.py`. Note
> that the advanced RAG features default to `false` in code even though a deployment `.env` often
> enables them — don't assume a feature is on; check the running config.

**Core Services:**
- `OLLAMA_HOST` - Ollama endpoint (default: http://localhost:11434)
- `OLLAMA_DEFAULT_MODEL=llama3.1:8b` - Default LLM
- `QDRANT_HOST`, `QDRANT_PORT` (6333), `QDRANT_GRPC_PORT` (6334), `QDRANT_COLLECTION_NAME=devops_docs`
- `REDIS_HOST`, `REDIS_PORT`, `REDIS_PASSWORD`, `REDIS_MAX_CONNECTIONS=50` - Memory & embedding cache
- `POSTGRES_HOST`, `POSTGRES_PORT` (5432), `POSTGRES_USER`, `POSTGRES_PASSWORD`, `POSTGRES_DB`, `POSTGRES_POOL_SIZE=10` - Analytics/auth/experiments DB
- `CHUNK_SIZE`, `CHUNK_OVERLAP`, `TOP_K_RESULTS` - RAG tuning
- `EMBEDDING_MODEL` (BAAI/bge-base-en-v1.5), `EMBEDDING_DIMENSION` (768), `EMBEDDING_DEVICE=auto`
- `EMBEDDING_CACHE_ENABLED=true`, `EMBEDDING_CACHE_TTL=3600`

**Multi-provider LLM:**
- `LLM_PROVIDER=ollama` - `ollama` (local) or `anthropic` (Claude API)
- `ANTHROPIC_API_KEY`, `ANTHROPIC_MODEL=claude-haiku-4-5-20251001` - Used when `LLM_PROVIDER=anthropic`

**Advanced RAG Features** (code defaults shown; often enabled via `.env`):
- `HYBRID_SEARCH_ENABLED=false` - BM25 + vector hybrid search (`HYBRID_SEARCH_ALPHA=0.5`, `HYBRID_RRF_K=60`)
- `RERANKER_ENABLED=false` - Cross-encoder reranking (`RERANKER_DEVICE=auto`, `RERANKER_TOP_K=5`, `RETRIEVAL_TOP_K=20`)
- `HYDE_ENABLED=false` - HyDE query expansion (`HYDE_MODEL=llama3.1:8b`)
- `CONVERSATION_CONTEXT_ENABLED=true` - Conversation-aware retrieval (`..._HISTORY_LIMIT=3`, `..._MAX_TERMS=10`)
- `CONVERSATION_SUMMARIZATION_ENABLED=false` - Tiered storage + auto-summarization (`..._THRESHOLD=10`)
- `CONTEXT_COMPRESSION_ENABLED=false` - Extract query-relevant passages from context
- `FEW_SHOT_ENABLED=true`, `CHAIN_OF_THOUGHT_ENABLED=true` - Prompt scaffolding
- `WEB_SEARCH_ENABLED=false` + `TAVILY_API_KEY` - Tavily fallback (`..._MIN_SCORE_THRESHOLD=0.4`, `..._INCLUDE_DOMAINS=...`)
- `SEMANTIC_CACHE_ENABLED=false` - Semantic response cache (`..._THRESHOLD=0.92`, `..._TTL=3600`)
- `OUTPUT_VALIDATION_ENABLED=true` - Hallucination detection (`..._MIN_CONFIDENCE=0.5`)

**Platform / Operations:**
- `AUTH_ENABLED=false` - JWT/API-key auth (`SESSION_EXPIRE_HOURS=24`, `API_KEY_PREFIX=rag_`)
- `QUERY_LOGGING_ENABLED=true` - Log queries to PostgreSQL
- `ANALYTICS_ENABLED=true` - Real-time metrics (`..._SHORT_WINDOW_SECONDS=300`, `..._ENDPOINT_PROTECTED=false`)
- `AB_TESTING_ENABLED=true` - A/B experiments (`..._AUTO_RECORD_METRICS=true`)
- `TRACING_ENABLED=false` - OpenTelemetry (`TRACING_EXPORTER=console|otlp`, `TRACING_OTLP_ENDPOINT`, `TRACING_SAMPLE_RATE=1.0`)
- `HEALTH_CHECK_VERBOSE=false` - Expose internal host/port details in `/api/health`
- `CORS_ORIGINS=http://localhost:3000` - Comma-separated allowed origins

### Docker
- **Production**: `docker-compose.yml` (Docker Hub images). Services: `ollama`, `qdrant`,
  `redis`, `postgres`, `backend`, `frontend`.
- **Development**: `docker-compose.dev.yml` (local builds, hot-reload)
- **GPU**: Ollama uses NVIDIA GPU via `runtime: nvidia`. The backend GPU image
  (`backend/Dockerfile.gpu` + `requirements-gpu.txt`, cu121) additionally runs embeddings and
  the cross-encoder reranker on GPU.

### Kubernetes
- `k8s/` holds production manifests (backend HPA/PDB, Qdrant/Postgres StatefulSets, Ollama on GPU
  nodes, ingress, network policies) composed with Kustomize overlays under `k8s/overlays/{dev,
  staging,production}`. Probes map to `/api/health/live` and `/api/health/ready`. See `k8s/README.md`.

## Modifying the System

### Adding Documentation Sources
1. Edit `scripts/download_docs.sh` to clone new repo
2. Add source to `doc_sources` dict in `scripts/ingest_docs.py`
3. Run `make download-docs && make ingest`

### Adding API Endpoints
```python
# 1. Define Pydantic models in models.py
class NewFeatureRequest(BaseModel):
    param: str

# 2. Add endpoint in main.py
@app.post("/api/new-feature")
async def new_feature(request: NewFeatureRequest):
    result = some_logic(request.param)
    return {"result": result}
```

### Adding Prompt Templates
```python
# Edit backend/app/templates.py - add to PROMPT_TEMPLATES list
{
    "id": "my-template",
    "category": "MyCategory",
    "title": "My Template Title",
    "description": "What this template does",
    "prompt": "The actual prompt text"
}
```

### Changing LLM Prompt
- Classic pipeline: modify `backend/app/rag.py` → `_build_messages()` method.
- Layered pipeline: modify `backend/app/retrieval/generators/ollama.py` → `_build_system_prompt()` / `_build_messages()`.
- Agentic pipeline: prompts are inline in `backend/app/agent.py` (`_plan`, `_reflect`, `_generate`, `_verify`).

## Debugging

```bash
make stats              # Check vector DB has data (points_count > 0)
make health             # Verify all services connected
docker exec ollama ollama list  # Check available models
docker compose restart backend  # Restart after code changes
```

### Performance Tuning
- **Faster**: smaller models (mistral:7b) or lower `TOP_K_RESULTS`
- **Better quality**: larger models or higher `TOP_K_RESULTS`
- **Memory issues**: reduce `OLLAMA_MAX_LOADED_MODELS` in docker-compose.yml

## Code Conventions

- Python: async/await for I/O, type hints, HTTPException for errors
- Module-level singletons for shared resources: `rag_pipeline`, `vector_store`,
  `conversation_storage`, `drift_detector`, `auth_service`, `get_metrics_collector()`,
  `get_llm_provider()`, `get_agentic_rag()`
- Imports inside methods/functions are intentional — they break circular imports and defer heavy
  ML model loads; preserve that pattern
- Background work uses `_create_background_task()` in `main.py` (keeps a reference, logs
  unhandled exceptions, auto-cleans). External calls (Ollama/Qdrant/Tavily) go through circuit breakers
- Configuration centralized in `config.py` via environment variables
- **Per-directory `AGENTS.md` files** document each part of the tree (purpose, key files, agent
  guidance) — read the local `AGENTS.md` before working in an unfamiliar directory

## Claude Code Agents

This repo includes 12 specialized AI/ML subagents in `.claude/agents/` for comprehensive code reviews:

| Agent | Focus |
|-------|-------|
| `ai-engineer` | AI system design, production deployment |
| `llm-architect` | LLM architecture, RAG optimization |
| `ml-engineer` | ML lifecycle, model serving |
| `mlops-engineer` | ML infrastructure, CI/CD |
| `prompt-engineer` | Prompt design, optimization |
| `data-engineer` | Data pipelines, ETL |
| `data-scientist` | Statistical analysis, modeling |
| `database-optimizer` | Query optimization, indexing |
| `nlp-engineer` | NLP pipelines, embeddings |
| `data-analyst` | Analytics, visualization |
| `postgres-pro` | Database administration |
| `machine-learning-engineer` | Inference, edge deployment |

**Usage:** Run parallel expert reviews with:
```
Launch 12 agents to review this project from different perspectives
```

*Credit: Agents from [awesome-claude-code-subagents](https://github.com/VoltAgent/awesome-claude-code-subagents) (categories/05-data-ai)*

## Known Limitations

- PostgreSQL is optional at runtime: if `init_db()` fails the app still serves chat, but query
  logging, analytics persistence, auth, and A/B testing degrade. Those are fire-and-forget paths
  that must never break a chat request.
- Authentication is **off by default** (`AUTH_ENABLED=false`); enable it before exposing publicly.
- Redis recent conversation history expires after 24h (summaries, when enabled, persist 7d).
- Vector search returns top 5 by default (configurable via `TOP_K_RESULTS`).
- Changing the embedding/reranker device requires a rebuild with the matching image
  (`Dockerfile` vs `Dockerfile.gpu`); `EMBEDDING_DEVICE=auto` falls back to CPU when no GPU is present.
