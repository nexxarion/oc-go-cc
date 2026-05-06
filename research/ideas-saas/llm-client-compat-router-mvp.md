# SaaS Idea: LLM Client Compatibility Router MVP

Status: research / out of current patch scope

## One-line idea

A hosted + local gateway that lets coding agents and Anthropic-shaped clients use OpenAI-compatible providers safely, with model-aware tool-call, streaming, reasoning, and resume-history normalization.

## Problem

Tools like Claude Code, OpenCode, Cursor-like agents, and custom agent runtimes are increasingly tied to specific response contracts:

- Anthropic SSE content block indexes must be exact.
- Tool calls stream differently across Qwen, DeepSeek, Kimi, GLM, MiniMax, Anthropic-compatible, and plain OpenAI-compatible providers.
- Some providers expose `reasoning_content`; Anthropic clients persist `thinking` blocks that require valid Anthropic signatures.
- Resume-history poisoning is hard to debug: one malformed thinking/tool block can break an otherwise useful long-running coding session.
- Teams want cheaper/faster model routing, but they do not want to hand-maintain dozens of provider quirks.

## MVP thesis

The product is not “just another proxy.” The value is a compatibility layer with a tested provider matrix and session-safety guarantees.

A useful MVP would provide:

1. A local gateway compatible with Anthropic Messages API clients.
2. A provider/model capability registry.
3. Model-aware normalization for:
   - streaming text deltas;
   - tool-only first responses;
   - parallel/multi tool calls;
   - reasoning/thinking fields;
   - stop reasons and usage blocks.
4. Claude Code resume-safety checks before persisting/replaying history.
5. Logs that explain why a request was transformed, stripped, retried, or routed.
6. A hosted dashboard for config, provider keys, routing rules, and compatibility reports.

## MVP scope

### Must-have

- Anthropic-compatible `/v1/messages` endpoint.
- Provider routing for 3-5 high-demand providers/models.
- Per-provider fixtures for streamed and non-streamed responses.
- Tool-call correctness tests:
  - first content block index starts at `0`;
  - no duplicate `content_block_start` for streamed argument chunks;
  - no unresolved tool calls after client disconnect/retry.
- Reasoning safety:
  - never emit invalid Anthropic `thinking` blocks;
  - preserve/replay provider reasoning only where the provider contract and client history model can support it coherently.
- Session scanner CLI: detect malformed persisted Claude Code histories and recommend safe repair steps.

### Nice-to-have

- Hosted policy editor for fallback chains and budgets.
- Cost/latency analytics per model and per tool turn.
- Replay debugger for a failed agent turn.
- Team-shared compatibility profiles.

## Explicitly out of scope for now

- Building a new IDE or agent client.
- Training/fine-tuning models.
- A full prompt-management platform.
- Enterprise SSO/compliance before product-market validation.
- Permanent deletion of provider reasoning as the final architecture. Dropping unsafe fields can be a short-term safety fallback, but the long-term goal is model-aware matching and synchronization.

## Differentiation

Most gateway products focus on simple routing, pricing, or OpenAI API compatibility. This idea focuses on agent-session correctness:

- Can the client resume tomorrow?
- Will Bash/tool calls complete?
- Are provider-specific reasoning fields handled without corrupting history?
- Can a team safely swap models under a coding agent without discovering edge cases in production?

## Validation plan

1. Use oc-go-cc as the local open-source gateway foundation.
2. Build a public compatibility matrix from real fixtures.
3. Test against Claude Code long-session resumes with tools enabled.
4. Ship a small hosted dashboard that manages local gateway config.
5. Find early users among developers running Claude Code/OpenCode against cheaper or specialized providers.

## Monetization hypothesis

- Free/open-source local gateway.
- Paid hosted control plane:
  - managed routing configs;
  - compatibility reports;
  - team dashboards;
  - session debugging/recovery tools;
  - SLA-tested provider profiles.

## Main risks

- Provider response formats change often.
- Client internals like Claude Code history behavior may change without notice.
- Some reasoning formats may be impossible to sync safely across incompatible APIs.
- Hosted key management raises trust/security requirements quickly.

## Next research questions

- Which providers truly require reasoning replay on tool-call continuation?
- Which clients persist Anthropic `thinking` blocks, and under what signature rules?
- Can the gateway maintain a shadow reasoning ledger outside client-visible history?
- What minimum compatibility test suite would make users trust a model/provider profile?
- Is the buyer an individual power user, an AI coding team, or a platform team managing many agent sessions?
