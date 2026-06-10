<!-- Parent: ../AGENTS.md -->
<!-- Generated: 2026-06-10 | Updated: 2026-06-10 -->

# strategies

## Purpose
Retrieval strategies that fetch candidate documents for a query. Implements the
`RetrievalStrategy` interface and returns a `RetrievalResult`. The hybrid strategy encapsulates
the full retrieval flow: hybrid (vector + BM25) or dense-only search → optional cross-encoder
reranking → score filtering → web-search fallback for low-quality results.

## Key Files

| File | Description |
|------|-------------|
| `hybrid.py` | `HybridRetrievalStrategy` — vector/BM25 search, reranking, min-score filtering, Tavily web-search fallback |

## For AI Agents

### Working In This Directory
- The strategy's `name` is computed from feature flags (`hybrid`, `hybrid_with_reranking`,
  `vector`, `vector_with_reranking`) — keep names stable as they appear in analytics.
- Vector store, reranker, and web searcher are lazy-loaded via `_get_*` helpers to avoid
  circular imports and honor `reranker_enabled` / `web_search_enabled` flags.
- Reranking over-fetches (`retrieval_top_k`) then reranks down to `reranker_top_k`, filtering by
  `min_rerank_score` (always keeps at least the top doc). `retrieve_async` offloads the sync
  path to a thread executor.
- Web-search fallback only triggers when result scores/counts fall below thresholds — preserve
  the `should_search` gating.

## Dependencies

### Internal
- `app.vectorstore`, `app.reranker`, `app.web_search`, `app.config`; base types from `../base.py`.

<!-- MANUAL: -->
