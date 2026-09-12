# codex-lmstudio Automation and Fork Notes

This directory contains fork-specific plans, operator guidance, and architecture notes for `dhar174/codex-lmstudio`.

The fork is an independent LM Studio/local-model focused Codex distribution. It is not required to continuously synchronize with `openai/codex`.

## Current direction

- [`FORK_POLICY.md`](FORK_POLICY.md): independent-fork maintenance rules and selective reuse policy for small `openai/codex` snippets.
- [`HOOKS.md`](HOOKS.md): backend lifecycle extension points for pre-model context augmentation and post-turn observers.
- [`MEMORY4AI.md`](MEMORY4AI.md): persistent-memory design using memory4ai through those hooks.
- [`PLAN.md`](PLAN.md): original implementation plan. Portions that require continuous upstream synchronization or Qdrant are superseded by the documents above and should be rewritten as the associated backlog issues are completed.

## Memory change

The earlier Qdrant MCP memory plan has been retired. Persistent memory will use memory4ai with automatic backend recall and capture hooks. The GUI may later expose settings/status, but it does not own memory execution or storage.
