<!-- Parent: ../AGENTS.md -->
<!-- Generated: 2026-06-10 | Updated: 2026-06-10 -->

# generators

## Purpose
Response generators that turn a query + retrieved context into a final answer. Implements the
`ResponseGenerator` interface and returns a `GenerationResult`. Supports both synchronous and
streaming generation. Currently backed by the local Ollama LLM.

## Key Files

| File | Description |
|------|-------------|
| `base.py` | `ResponseGenerator` ABC + `GenerationResult` dataclass |
| `ollama.py` | `OllamaGenerator` — builds DevOps system prompt + messages, calls Ollama (sync + streaming) |

## For AI Agents

### Working In This Directory
- Implement `name`, `generate()`, and `generate_stream()` for new providers; lazy-load the
  client (Ollama is imported on first use).
- The system prompt in `ollama.py` (`_build_system_prompt`) defines the assistant persona and
  DevOps domain scope; conversation history is folded into the message list via `_build_messages`.
- Streaming yields chunk dicts consumed by the orchestrator and re-emitted as SSE by the API.

## Dependencies

### Internal
- `app.config` (default model, generation settings); base types from `../base.py` equivalents.

### External
- `ollama` Python client.

<!-- MANUAL: -->
