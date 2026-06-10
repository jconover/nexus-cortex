<!-- Parent: ../AGENTS.md -->
<!-- Generated: 2026-06-10 | Updated: 2026-06-10 -->

# expanders

## Purpose
Query-expansion strategies for the retrieval pipeline. An expander rewrites/enriches the user
query before search to improve recall — e.g. resolving pronouns from conversation history or
generating a hypothetical answer document (HyDE). All conform to the `QueryExpander` interface
and return an `ExpansionResult`.

## Key Files

| File | Description |
|------|-------------|
| `base.py` | `QueryExpander` ABC + `ExpansionResult` dataclass (original/expanded query, terms, timing, metadata) |
| `conversation.py` | Conversation-aware expander — resolves references using prior messages |
| `hyde.py` | HyDE expander — generates a hypothetical document to embed instead of the raw query |
| `composite.py` | Chains multiple expanders together |

## For AI Agents

### Working In This Directory
- Implement `name`, `is_available()`, and `expand_async()` (and the sync variant if relevant).
- Expanders must fail soft: return an unexpanded `ExpansionResult` (or set `error`/`skip_reason`)
  rather than raising — the orchestrator continues with the original query on failure.
- Set `expanded=True` only when the query actually changed; populate `context_terms` and
  `metadata` for observability.

## Dependencies

### Internal
- `app.config` (feature flags), `app.conversation_context`, `app.query_expansion` (HyDE),
  `app.llm_provider` for LLM-backed expansion.

<!-- MANUAL: -->
