<!-- Parent: ../AGENTS.md -->
<!-- Generated: 2026-06-10 | Updated: 2026-06-10 -->

# retrieval

## Purpose
A layered, pluggable RAG abstraction that decomposes the pipeline into three composable
roles — **query expanders**, **retrieval strategies**, and **response generators** — wired
together by an orchestrator. This is the cleaner, refactored alternative to the monolithic
`app/rag.py`, with well-defined dataclass result types (`ExpansionResult`, `RetrievalResult`,
`GenerationResult`) and `to_dict()` metadata for observability.

## Key Files

| File | Description |
|------|-------------|
| `pipeline.py` | `RAGPipelineOrchestrator` — runs expand → retrieve → generate (sync + streaming); `create_default_orchestrator()` wires expanders from feature flags |
| `base.py` | `RetrievalStrategy` ABC + `RetrievalResult` dataclass (scores, timings, cache-hit, metadata) |

## Subdirectories

| Directory | Purpose |
|-----------|---------|
| `expanders/` | Query-expansion strategies (conversation, HyDE, composite) (see `expanders/AGENTS.md`) |
| `strategies/` | Retrieval strategies (hybrid vector+BM25) (see `strategies/AGENTS.md`) |
| `generators/` | Response generators (Ollama) (see `generators/AGENTS.md`) |

## For AI Agents

### Working In This Directory
- Add a new capability by implementing the relevant ABC (`QueryExpander`,
  `RetrievalStrategy`, `ResponseGenerator`) and registering it with the orchestrator —
  prefer this over extending `rag.py`.
- Expanders are applied in sequence; each must implement `is_available()` and
  `expand_async()` and fail soft (the orchestrator logs and continues on expander errors).
- The default orchestrator enables expanders based on `settings`
  (`conversation_context_enabled`, `hyde_enabled`). Heavy components are lazy-loaded to avoid
  circular imports.
- Result dataclasses are the contract between stages — keep `to_dict()` fields stable since
  they surface in API metadata and analytics.

### Common Patterns
- Every stage returns a rich dataclass with timing + metadata, never bare values.
- `retrieve_async` typically offloads sync retrieval to a thread executor.

## Dependencies

### Internal
- `app.vectorstore`, `app.reranker`, `app.web_search`, `app.config`, and the
  expander/generator/strategy subpackages.

### External
- langchain-core (Document), asyncio.

<!-- MANUAL: -->
