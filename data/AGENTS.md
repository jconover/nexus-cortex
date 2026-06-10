<!-- Parent: ../AGENTS.md -->
<!-- Generated: 2026-06-10 | Updated: 2026-06-10 -->

# data

## Purpose
Holds the documentation corpus and supporting data for the RAG system: cloned documentation
sources to be ingested, user-uploaded custom documents, and evaluation datasets. Most contents
are generated/downloaded artifacts rather than source code — `data/docs/` is populated by
`make download-docs`.

## Subdirectories

| Directory | Purpose |
|-----------|---------|
| `docs/` | Ingestion corpus — documentation repos cloned by `scripts/download_docs.sh` (gitignored / generated; empty until populated) |
| `custom/` | User-supplied custom documents to index (see `custom/README.md`) |
| `eval/` | Evaluation datasets (`devops_eval_set.json`) used by benchmarking/RAGAS scoring |

## For AI Agents

### Working In This Directory
- Treat `docs/` as a generated cache: populate with `make download-docs`, refresh with
  `make update-docs`. Do not hand-curate large content here or commit downloaded corpora.
- `eval/devops_eval_set.json` is consumed by `scripts/benchmark_rag.py` and the backend's RAG
  evaluation (`backend/app/evaluation.py` / `rag_eval.py`); keep its schema stable when editing.

## Dependencies

### Internal
- Populated and consumed by `scripts/` (download/ingest) and the backend evaluation modules.

<!-- MANUAL: -->
