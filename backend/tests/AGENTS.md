<!-- Parent: ../AGENTS.md -->
<!-- Generated: 2026-06-10 | Updated: 2026-06-10 -->

# tests

## Purpose
Pytest suite covering the backend API surface and key RAG components. Uses shared fixtures
from `conftest.py`.

## Key Files

| File | Description |
|------|-------------|
| `conftest.py` | Shared pytest fixtures and test configuration |
| `test_api.py` | FastAPI endpoint tests |
| `test_rag.py` | RAG pipeline behavior tests |
| `test_rag_validation.py` | RAG input/output validation tests |
| `test_output_validation.py` | Hallucination/output-validation tests |
| `test_semantic_cache.py` | Semantic response cache tests |
| `test_analytics.py` | Analytics collector tests |
| `test_templates.py` | Prompt template rendering/validation tests |

## For AI Agents

### Working In This Directory
- Run from `backend/`: `pytest` (configuration in `backend/pytest.ini`).
- External services (Ollama, Qdrant, Redis, PostgreSQL) should be mocked/faked via fixtures —
  do not require a live stack for unit tests.
- When adding a feature to `app/`, add or extend the corresponding `test_*.py` module here.

### Common Patterns
- Tests mirror the module they exercise (`test_<module>.py`).

## Dependencies

### Internal
- The `app` package (imported as `app.*`).

### External
- pytest, FastAPI TestClient / httpx.

<!-- MANUAL: -->
