# Codex LM Studio Fork Policy

## Independent fork

`dhar174/codex-lmstudio` is an independent LM Studio-focused Codex fork. Its own maintained branch history, behavior, compatibility requirements, and release decisions are authoritative for this project.

The repository is not required to continuously rebase, merge, or otherwise synchronize with `openai/codex`. Upstream may be consulted as a useful implementation reference, but it is not a synchronization contract.

## Selective reuse from `openai/codex`

Small, narrowly scoped snippets or implementation ideas from `openai/codex` may be adapted into this fork when all of the following are true:

1. The reused code directly solves the issue currently being worked on or updates a capability that is explicitly in scope for that issue.
2. The change preserves this fork's LM Studio/local-model behavior and does not break existing fork-specific capabilities.
3. Integration requires only a minor amount of adaptation, technical debt, dependency creep, schema churn, or architectural work beyond the issue's existing scope.
4. The reused code does not create an ongoing dependency on upstream synchronization or require a broad migration toward a newer upstream architecture.
5. Relevant tests and compatibility checks are run for the affected fork behavior.

Prefer local adaptation of the smallest useful implementation over wholesale merges, broad cherry-picks, or importing unrelated refactors.

If a useful upstream change would require substantial refactoring, new cross-cutting dependencies, protocol or schema migration, significant architectural churn, or work outside the active issue, stop and create a separate issue rather than quietly expanding scope.

For non-trivial copied code, preserve useful provenance in the commit message, PR description, or nearby source comment as appropriate, and continue to comply with the repository's Apache-2.0 licensing obligations.

## Memory architecture

Long-term agent memory for this fork is based on `memory4ai`, integrated through fork-native lifecycle hooks rather than a Qdrant MCP memory server.

The intended architecture is:

- a pre-turn / pre-model-request hook for bounded relevant-memory recall and ephemeral context injection;
- a post-turn / response-completion hook for asynchronous memory capture;
- fail-open behavior so memory failures never abort a valid coding turn;
- one backend implementation shared by the CLI and App Server, allowing `codex-lmstudio-ui` to inherit memory behavior without implementing a second memory engine.

Memory context must obey the repository's existing bounded-context and `ContextualUserFragment` rules.
