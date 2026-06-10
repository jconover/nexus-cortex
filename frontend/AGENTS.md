<!-- Parent: ../AGENTS.md -->
<!-- Generated: 2026-06-10 | Updated: 2026-06-10 -->

# frontend

## Purpose
React single-page chat UI for the DevOps AI Assistant. Talks to the FastAPI backend for
chat (standard and streaming/SSE), model selection, stats, health, prompt templates, and
document upload. Served via nginx in production.

## Key Files

| File | Description |
|------|-------------|
| `package.json` | Dependencies (React, axios, react-markdown, remark-gfm) and scripts |
| `Dockerfile` | Multi-stage build → static assets served by nginx |
| `nginx.conf` | nginx config for serving the SPA and proxying to the backend |

## Subdirectories

| Directory | Purpose |
|-----------|---------|
| `src/` | Application source (App component, styles) (see `src/AGENTS.md`) |
| `public/` | Static HTML shell (`index.html`) |

## For AI Agents

### Working In This Directory
- The backend base URL comes from `REACT_APP_API_URL` (defaults to `http://localhost:8000`).
- This is a Create React App-style project; standard `npm start` / `npm run build` apply.

### Testing Requirements
- No automated frontend tests currently; verify manually against a running backend.

### Common Patterns
- Single `App.js` component holds chat state (messages, model, session, streaming toggle,
  theme persisted to localStorage). Streaming responses are consumed as Server-Sent Events.

## Dependencies

### Internal
- Backend REST API at `/api/*` (chat, chat/stream, models, stats, health, templates, upload)

### External
- React, axios, react-markdown, remark-gfm

<!-- MANUAL: -->
