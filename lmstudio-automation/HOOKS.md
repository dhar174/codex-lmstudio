# Codex LM Studio Lifecycle Hooks

## Purpose

The fork should provide small, stable lifecycle extension points for local integrations that need to observe or augment agent turns without becoming model-visible tools.

The first consumer is `memory4ai`, but the hook mechanism should remain generic.

## Initial lifecycle points

### Pre-model context hook

Runs after the user turn and normal Codex context have been assembled, but before the model request is emitted.

A hook may return bounded ephemeral context fragments to add to the request. It must not rewrite existing conversation history.

Primary initial use: relevant-memory recall from `memory4ai`.

Requirements:

- context additions must use the repository's normal context abstractions, including `ContextualUserFragment` where applicable;
- every injected fragment must have a hard size/count bound;
- deterministic ordering when multiple hooks are configured;
- configurable timeout;
- fail-open behavior by default for optional integrations;
- hook failures must be diagnosable without leaking secrets or unbounded prompt content.

### Post-turn completion hook

Runs after a turn has reached a completed terminal state and the final user/assistant exchange is available.

Primary initial use: asynchronous `memory4ai` capture.

Requirements:

- may run detached/asynchronously when the caller does not require a result;
- must not delay delivery of an otherwise completed response;
- receives an immutable/bounded view of the completed turn rather than mutable agent state;
- retry behavior, if any, must be bounded;
- failures are logged/observable but do not convert a successful Codex turn into a failed turn.

## Product boundaries

- Hooks are backend infrastructure shared by CLI and App Server.
- The model does not need to call hooks explicitly.
- App Server clients such as `codex-lmstudio-ui` inherit hook behavior automatically.
- A GUI may later expose hook/memory configuration and status, but must not duplicate hook execution or memory storage.
- Hook APIs should be narrow enough that adding a new integration does not require plumbing provider-specific code throughout `codex-core`.

## Memory4ai

See GitHub issue #11 for the initial memory4ai consumer and `FORK_POLICY.md` for the fork's maintenance and selective-upstream-reuse rules.
