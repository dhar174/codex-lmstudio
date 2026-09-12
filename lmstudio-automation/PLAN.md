# Codex LM Studio Integration Plan

## Objective

Maintain two independent Codex installations:

- `codex`: the unmodified official installation using ChatGPT Plus/OpenAI and `~/.codex`
- `codex-lmstudio`: a separately installed fork build using LM Studio, its own `CODEX_HOME`, local MCP servers, and Qdrant

The fork's own maintained branch history is the authoritative product baseline. It must rebuild safely, publish verified Linux artifacts, and never disrupt the official installation. See `FORK_POLICY.md`, `HOOKS.md`, and `MEMORY4AI.md` for the current architecture.

## Environment

- Fork: `dhar174/codex-lmstudio`
- Upstream: `openai/codex`
- Work branch: `codex/lmstudio-automation`
- Host: Windows with WSL2 Ubuntu
- LM Studio endpoint: `http://127.0.0.1:1234/v1`
- Initial model: `devstral-small-2-24b-instruct-2512`
- Qdrant endpoint: `http://127.0.0.1:6333`
- Qdrant MCP launcher: `uvx mcp-server-qdrant`

## Known compatibility concerns

### MCP namespace tools

Codex can expose MCP tools through the Responses API as namespace tool definitions. LM Studio currently ignores unsupported namespace tool entries, preventing local models from receiving those MCP tools.

If equivalent support is not already present upstream, the fork will add a provider-specific compatibility mode that flattens each namespaced tool into a deterministic ordinary function name and reverses returned calls back to the original MCP server/tool identity.

### Devstral message template

Devstral's default Jinja template can reject adjacent user messages or converted developer/system messages. This is separate from MCP namespace handling. The repository will document a tested LM Studio template override, but the Rust compatibility patch will not attempt to solve prompt-template behavior.

## Upstream PR #28271 integration decision

OpenAI Codex PR #28271 (`kotakem/issue-26234-flatten-mcp-tools`) already implements the same provider-specific namespace-tool compatibility direction planned here.

Repository references:

- Vendor branch: `vendor/openai-pr-28271`
- Imported head: `0b7b2436bb2b6b4f5eeca64ab72a9ee8cee5844d`
- Draft comparison PR: `#1`

The exact upstream PR head is preserved as a vendor reference, but it will not become this fork's long-lived base. The vendor branch is five commits ahead of its merge base and substantially behind the fork's current `main`; GitHub reports that it does not merge cleanly into `codex/lmstudio-automation`.

The implementation strategy is therefore:

1. Keep imported/upstream branches or PRs only as historical/provenance references where useful.
2. Treat future upstream reuse as deliberate, issue-scoped maintenance rather than a required workflow.

Unresolved upstream review findings that must be addressed:

- reject or disambiguate canonical flat-name collisions before exposing flattened MCP functions
- resolve flat calls to one unambiguous runtime instead of first-match hash-map lookup
- preserve canonical runtime names for pre/post tool hooks, especially `spawn_agent`/`Agent`
- keep standalone web and image tools available for Responses Lite providers that disable namespace wrappers
- preserve Responses API name-length limits for flattened dynamic tools

Current-main observations:

- `ProviderCapabilities` already includes `namespace_tools`, but current configured providers do not yet resolve it from `ModelProviderInfo`.
- `ModelProviderInfo` does not yet expose a `namespace_tools` configuration field.
- `responses_api.rs` still serializes namespace tools without a flattening helper.
- `ToolRegistry` still performs exact-name lookup only.
- current MCP normalization already preserves raw identities and hashes model-visible namespace/tool collisions; the forward-port should build on that rather than duplicate it.

## Phase 1: Current-source discovery

- [x] Confirm the current code paths for provider capabilities and provider configuration.
- [x] Confirm the current Responses API tool serialization path.
- [x] Confirm MCP tool registration, model-visible naming, and call dispatch paths.
- [x] Search current upstream source, issues, and pull requests for equivalent compatibility work.
- [x] Document the minimal design, affected crates, tests, and patch-retirement conditions.
- [ ] Confirm current build, formatting, schema-generation, and release conventions.

### Initial source findings

- MCP server connections and tool aggregation are owned by `codex-rs/codex-mcp/src/connection_manager.rs`.
- Model-visible MCP namespace and tool-name normalization is owned by `codex-rs/codex-mcp/src/tools.rs`.
- Current `ToolInfo` already preserves raw server/tool identity separately from model-visible `callable_namespace` and `callable_name`.
- Provider capabilities are owned by `codex-rs/model-provider/src/provider.rs`.
- User-configurable provider metadata is owned by `codex-rs/model-provider-info/src/lib.rs`.
- Responses namespace serialization is owned by `codex-rs/tools/src/responses_api.rs`.
- Runtime resolution and hook dispatch are owned by `codex-rs/core/src/tools/registry.rs`.
- The implementation should extend existing abstractions rather than introduce a second parallel naming system.

## Phase 2: Provider-specific namespace compatibility

- [ ] Add a provider capability/config option using current project conventions.
- [ ] Preserve namespace representation for providers that support it.
- [ ] Flatten namespace tools only for providers that do not support it.
- [ ] Use deterministic, reversible names within provider limits.
- [ ] Reject collisions, malformed names, and ambiguous mappings safely.
- [ ] Reverse-map returned flat calls to the original MCP server and tool.
- [ ] Preserve canonical runtime names for hooks and telemetry after reverse mapping.
- [ ] Preserve arguments, call IDs, streaming events, parallel calls, and ordinary functions.
- [ ] Preserve standalone Responses Lite web/image tools when namespaces are unavailable.
- [ ] Preserve existing defaults and OpenAI-provider behavior.
- [ ] Update the generated config schema when required.

### Required Rust tests

- [ ] Namespace-capable provider retains namespace representation.
- [ ] Namespace-incapable provider receives flattened ordinary functions.
- [ ] Similar names from separate MCP servers remain distinct.
- [ ] Canonical flat-name collisions fail safely or receive deterministic disambiguation.
- [ ] Arguments and schemas remain unchanged.
- [ ] Returned flat calls dispatch to the correct server/tool.
- [ ] Invalid and ambiguous names fail safely.
- [ ] Hook matching receives the canonical runtime name after flat resolution.
- [ ] Streaming tool calls preserve names and call IDs.
- [ ] Parallel tool calls continue to work.
- [ ] Ordinary functions remain unchanged.
- [ ] Responses Lite standalone web/image tools remain visible.
- [ ] Default and OpenAI-provider behavior remains unchanged.

## Phase 3: Isolated installation tooling

Create `lmstudio-automation/` assets for:

- [ ] `bootstrap.sh`
- [ ] `build-local.sh`
- [ ] `update-local.sh`
- [ ] `install-release.sh`
- [ ] `verify-install.sh`
- [ ] `rollback.sh`
- [ ] `uninstall.sh`
- [ ] `render-config.py`
- [ ] `env.example`
- [ ] `config.toml.template`
- [ ] `codex-lmstudio` launcher
- [ ] optional user-level systemd service and timer

Default local layout:

```text
~/.local/lib/codex-lmstudio/codex
~/.local/lib/codex-lmstudio/codex.previous
~/.local/bin/codex-lmstudio
~/.local/bin/update-codex-lmstudio
~/.codex-lmstudio/config.toml
~/.config/codex-lmstudio/env
~/.local/state/codex-lmstudio/
```

The launcher must set a separate `CODEX_HOME` and must never modify the official `codex` executable or `~/.codex`.

## Phase 4: LM Studio and Qdrant configuration

- [ ] Render configuration from environment variables without hardcoded user paths.
- [ ] Configure the LM Studio Responses provider.
- [ ] Configure Qdrant MCP over stdio using `uvx`.
- [ ] Default persistent Qdrant to localhost Docker at port 6333.
- [ ] Allow future mutually exclusive `QDRANT_LOCAL_PATH` support.
- [ ] Verify LM Studio endpoint/model, Qdrant endpoint, config parsing, MCP startup, and tool discovery.
- [ ] Document the Devstral Jinja-template override separately.

## Phase 5: Upstream synchronization and releases

Add GitHub Actions workflows for:

- [ ] focused Rust tests and formatting checks
- [ ] Linux x86_64 release builds
- [ ] SHA-256 checksums and build metadata
- [ ] rolling prerelease publication as `lmstudio-latest`
- [ ] artifact publication only after successful validation

Repository rules must be respected. Workflows must not assume arbitrary branch creation or unsafe force pushes are allowed.

## Phase 6: Documentation and validation

- [ ] Add operator-focused `lmstudio-automation/README.md`.
- [ ] Document architecture, install, configuration, updates, rollback, uninstall, Qdrant, LM Studio, Devstral, and troubleshooting.
- [ ] Run `just fmt`.
- [ ] Run affected crate tests with `just test -p ...`.
- [ ] Run applicable scoped `just fix -p ...`.
- [ ] Run schema generation when config types change.
- [ ] Run Bash syntax and ShellCheck validation.
- [ ] Run Python renderer checks.
- [ ] Review workflow syntax and permissions.
- [ ] Scan changed files for secrets.
- [ ] Review the complete diff and security implications.

## Safety requirements

- Never overwrite, wrap, or remove the official `codex` command.
- Never modify or delete `~/.codex`.
- Never commit credentials, generated configurations, environment files, or binaries.
- Bind Qdrant to localhost by default.
- Never delete Qdrant collections, containers, volumes, or storage.
- Never install a failed or unverified build.
- Preserve the previous custom binary for rollback.
- Keep custom automation isolated and minimize divergence from upstream.

## Completion criteria

- `codex` remains the official installation using ChatGPT Plus/OpenAI.
- `codex-lmstudio` uses a separate binary, config directory, sessions, and logs.
- LM Studio receives callable MCP functions when namespace tools are unsupported.
- Returned calls dispatch to the correct MCP server and tool.
- Qdrant `qdrant-store` and `qdrant-find` are visible and callable through Codex with a compatible local model.
- The work is delivered through a reviewed pull request to `main`; nothing merges automatically.
