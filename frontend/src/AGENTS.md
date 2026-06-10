<!-- Parent: ../AGENTS.md -->
<!-- Generated: 2026-06-10 | Updated: 2026-06-10 -->

# src

## Purpose
React application source for the chat UI. A single primary component manages chat state and
all interactions with the backend API.

## Key Files

| File | Description |
|------|-------------|
| `index.js` | React entrypoint; mounts `<App />` |
| `App.js` | Main chat component: messages, model selection, session ID, streaming toggle, theme, prompt templates, document upload |
| `App.css` | Component styles (incl. light/dark theme) |
| `index.css` | Global base styles |

## For AI Agents

### Working In This Directory
- Backend base URL comes from `process.env.REACT_APP_API_URL` (default `http://localhost:8000`).
- Streaming chat consumes Server-Sent Events from `/api/chat/stream`; non-streaming uses
  `/api/chat`. The streaming preference and theme are persisted in `localStorage`.
- Markdown answers are rendered with `react-markdown` + `remark-gfm`.

### Common Patterns
- State held via `useState`/`useRef` in `App.js`; side effects (fetch models/stats/health/
  templates, autoscroll, persistence) via `useEffect`.

## Dependencies

### Internal
- Backend REST endpoints: `/api/chat`, `/api/chat/stream`, `/api/models`, `/api/stats`,
  `/api/health`, `/api/templates`, `/api/upload`.

### External
- React, axios, react-markdown, remark-gfm.

<!-- MANUAL: -->
