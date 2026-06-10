<!-- Parent: ../AGENTS.md -->
<!-- Generated: 2026-06-10 | Updated: 2026-06-10 -->

# n8n-workflows

## Purpose
Exported n8n automation workflows that schedule documentation updates and re-ingestion so the
RAG index stays fresh without manual intervention. Importable into an n8n instance. See the
local `README.md` for setup details.

## Key Files

| File | Description |
|------|-------------|
| `README.md` | How to import and configure the workflows in n8n |
| `daily-doc-update.json` | Daily scheduled documentation update/ingest workflow |
| `weekly-doc-update.json` | Weekly scheduled documentation update/ingest workflow |

## For AI Agents

### Working In This Directory
- These are JSON exports of n8n workflows, not application code. Edit in the n8n UI and
  re-export rather than hand-editing node graphs where possible.
- Workflows typically trigger the same operations as `make update-docs` / `make ingest`.

## Dependencies

### Internal
- Documentation ingestion pipeline in `scripts/` and the backend ingest endpoints.

### External
- A running n8n instance. Related MCP integration is documented in root `MCP_N8N_INTEGRATION.md`.

<!-- MANUAL: -->
