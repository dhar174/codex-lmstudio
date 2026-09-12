# memory4ai Integration Direction

`codex-lmstudio` will use `memory4ai` for persistent agent memory instead of the previously planned Qdrant MCP memory server.

## Design

- Recall is automatic through a backend pre-model lifecycle hook.
- Capture is automatic through a backend post-turn lifecycle hook.
- The local model is not required to decide when to call a memory tool.
- CLI and App Server share the same memory implementation.
- `codex-lmstudio-ui` may expose status, settings, or diagnostics later, but memory ownership stays in the backend.
- Memory failures are fail-open and never invalidate an otherwise successful Codex turn.

## Recall

The recall hook should query memory4ai before inference and inject only relevant, bounded, ephemeral context.

Preferred behavior:

- hybrid lexical/BM25 + embedding retrieval when available;
- graceful fallback to text-only search when embeddings are unavailable;
- configurable relevance threshold;
- configurable result count and hard context-size cap;
- no conversation-history rewriting;
- use normal Codex context abstractions for injected fragments.

## Capture

After a completed turn, capture the useful user/assistant exchange asynchronously.

Filtering should suppress obvious memory pollution such as:

- duplicate or near-duplicate captures;
- trivial completion acknowledgements;
- retry/continue/restart chatter;
- repetitive automation-loop messages;
- failed/interrupted turns that do not contain useful durable information.

## Storage

Use an explicit, fork-owned memory4ai configuration/database path. Do not rely on an implicit current-working-directory database path.

The exact path should be configurable and should live under the custom `CODEX_HOME` or another dedicated `codex-lmstudio` state directory by default.

## Diagnostics

Provide bounded, opt-in diagnostics for recall/capture decisions and failures. Debug logging must avoid credentials, secrets, and unbounded prompt/response dumps.

## Tracking

- GitHub issue #11 tracks the memory4ai integration.
- `HOOKS.md` defines the generic lifecycle-hook architecture.
- `FORK_POLICY.md` defines independent-fork and selective-upstream-reuse rules.
