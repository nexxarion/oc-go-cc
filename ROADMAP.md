# oc-go-cc Roadmap

## Deferred: model-aware reasoning/tool synchronization

The current fork patch favors Claude Code resume safety by dropping unsigned OpenAI-compatible `reasoning_content` instead of emitting invalid Anthropic `thinking` blocks. That is a safe short-term guardrail, not the desired long-term architecture.

Long-term direction:

- Build a provider/model capability matrix for response shapes across Qwen, DeepSeek, Kimi, GLM, MiniMax, Anthropic-compatible providers, and plain OpenAI-compatible providers.
- Compare each model's streaming and non-streaming formats for:
  - `reasoning_content` / thinking deltas;
  - tool call IDs, indexes, argument deltas, finish reasons;
  - whether prior reasoning must be replayed on later tool turns;
  - whether the provider accepts/needs placeholder reasoning on tool-call continuations.
- Add a model-aware normalization layer instead of one global transform path.
- Preserve reasoning only when it can round-trip coherently for that provider and client without poisoning Anthropic/Claude Code history.
- Keep Anthropic `thinking` blocks reserved for data that has valid Anthropic signatures, unless a client-specific compatibility mode explicitly strips them before resume replay.
- Add regression fixtures per provider/model for:
  - text-only responses;
  - tool-only first responses starting at content index `0`;
  - text → tool, reasoning → text, reasoning → tool;
  - multiple/parallel tool calls;
  - resumed Claude Code sessions with persisted history.

Acceptance target: no dropped tool calls, no stuck Bash calls, no invalid `thinking` signatures on Claude Code resume, and provider-specific reasoning replay only where the upstream API contract actually supports it.
