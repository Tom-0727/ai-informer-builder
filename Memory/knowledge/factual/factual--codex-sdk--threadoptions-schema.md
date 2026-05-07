---
id: kn.codex-sdk.factual.threadoptions-schema
kind: factual
summary: Codex SDK ThreadOptions has six top-level keys (name, description, model, model_reasoning_effort, sandbox_mode, developer_instructions) — no web_fetch or tools toggle; behavioral bounds are encoded in developer_instructions prose plus sandbox_mode.
last_edited_at: 2026-05-07
status: active
tags: [codex-sdk, schema, factual, agent-runtime]
---

## Core Content

The Codex SDK ThreadOptions surface (TypeScript/JavaScript shape, validated against SDK ~0.128.0) exposes exactly six top-level keys. Source: SDK `dist/index.js` plus episodes ep.20260506T050000Z (.toml authoring) and ep.20260506T230500Z (sandbox-network coupling).

- `name` — string. Identifier of the agent. Maps to the CLI agent name passed via `--agent <name>` or the equivalent in agent-launch invocations.
- `description` — string. Human-readable description. Surfaced in agent listings; not consumed by the model at runtime.
- `model` — string. Model id (e.g. `gpt-5-codex`, `gpt-4.1`). Maps to the CLI `--model <id>` flag.
- `model_reasoning_effort` — enum string (`low | medium | high`). Maps to the CLI `--config model_reasoning_effort=<level>`. Controls reasoning-token budget per turn.
- `sandbox_mode` — enum string (`read-only | workspace-write | danger-full-access`). Maps to the CLI `--sandbox <mode>` flag. Selects the active sandbox profile for the agent run.
- `developer_instructions` — string (free-form prose). Maps to the system/developer prompt prepended to the agent's context. Behavioral bounds — what the agent should and should not do, scope, output shape — live here, not in a separate options field.

Notable absences (validated by enumerating the SDK's emitted CLI invocation against an enabled-everything ThreadOptions):

- No `web_fetch` toggle. Web fetch is implicit in the active sandbox profile, not an independent option.
- No `tools` allowlist. Tool selection is bound to the agent's `name` (the underlying agent .toml declares the tool set) and not parameterized at ThreadOptions level.
- No `network_access` global. The SDK exposes `networkAccessEnabled` (camelCase JS field), which translates to a SCOPED config `--config sandbox_workspace_write.network_access=true`. That scoped key only takes effect when `sandbox_mode` is `workspace-write` — the dependency is structural, not an option ordering accident.

SDK-to-CLI mapping summary (snake_case JS fields → CLI):
- `name` → `--agent <name>`
- `model` → `--model <id>`
- `model_reasoning_effort` → `--config model_reasoning_effort=<level>`
- `sandbox_mode` → `--sandbox <mode>`
- `networkAccessEnabled: true` (camelCase JS, separate from the six snake_case keys) → `--config sandbox_workspace_write.network_access=true`
- `developer_instructions` → injected as the developer/system prompt; no separate flag.

## Limitations

- Validated against SDK ~0.128.0. Future SDK refactors may merge scoped configs into a single global key (e.g. a top-level `network_access`), which would invalidate the scoped-mapping detail above.
- Not yet validated against any ThreadOptions extension surface — if the SDK exposes hooks, callbacks, or middleware in a future version, the "exactly six keys" claim would no longer be exhaustive.
- The mapping was reverse-engineered from `dist/index.js`; the SDK README at this version does not document the CLI translation. A future README update may surface or change details.

## Optional Notes

- Provenance: ep.20260506T050000Z first surfaced the schema during subagent .toml authoring (initial discovery of the six keys); ep.20260506T230500Z confirmed the scoped sandbox-network coupling during the network-fix episode.
- Cross-reference: kn.codex-sdk.heuristic.sandbox-network-requires-explicit-mode — the operational consequence of the structural dependency between `networkAccessEnabled` and `sandbox_mode`.
