<!-- Parent: ../AGENTS.md -->
<!-- Generated: 2026-06-10 | Updated: 2026-06-10 -->

# repositories

## Purpose
Repository-pattern data-access layer that abstracts persistence behind a consistent async
interface with context management, structured errors, and tracing context. Provides a clean
seam between business logic and the underlying stores (PostgreSQL via async SQLAlchemy, Qdrant).

## Key Files

| File | Description |
|------|-------------|
| `base.py` | `BaseRepository[T]` (async context mgmt, error types, logging) and `AsyncSessionRepository` (SQLAlchemy session lifecycle + `transaction()`) |
| `conversation.py` | Conversation-history repository |
| `query_log.py` | Query-log repository (analytics writes/reads) |
| `vector.py` | Vector-search repository wrapping Qdrant operations |

## For AI Agents

### Working In This Directory
- Subclass `AsyncSessionRepository` for PostgreSQL-backed repositories; use the
  `async with repo:` context manager (or `transaction()`) so connections/sessions are cleaned up.
- Raise the typed errors from `base.py` (`NotFoundError`, `QueryError`, `ConnectionError`,
  `ValidationError`, `DuplicateError`) instead of leaking raw driver exceptions.
- Session factory is lazily resolved via `app.database.get_session_factory` to avoid circular
  imports — keep that indirection.

### Common Patterns
- Generic `BaseRepository[T]` parameterized by entity type; `RepositoryContext` carries
  operation/user/session/trace IDs for logging and audit.

## Dependencies

### Internal
- `app.database` (session factory), `app.db_models` (ORM entities), `app.vectorstore` (Qdrant).

### External
- SQLAlchemy (async), qdrant-client.

<!-- MANUAL: -->
