<!-- Parent: ../AGENTS.md -->
<!-- Generated: 2026-06-10 | Updated: 2026-06-10 -->

# migrations

## Purpose
SQL migrations for the PostgreSQL analytics database. Currently focused on partitioning the
high-volume query-log table for scalable analytics.

## Key Files

| File | Description |
|------|-------------|
| `partition_query_logs.sql` | Converts/partitions the `query_logs` table (pairs with `scripts/create_partitions.py`) |

## For AI Agents

### Working In This Directory
- Migrations are applied against the PostgreSQL instance defined in `.env` / `database.py`.
- Schema here must stay consistent with the ORM models in `backend/app/db_models.py`.
- `scripts/create_partitions.py` is the Python companion that manages partition creation.

<!-- MANUAL: -->
