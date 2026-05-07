---
id: kn.codex-sdk.heuristic.sandbox-network-requires-explicit-mode
kind: heuristic
summary: Codex SDK networkAccessEnabled true only engages when sandboxMode is set to workspace-write — the SDK option translates to a scoped CLI config that is silent unless the matching profile is selected.
last_edited_at: 2026-05-07
status: active
tags: [codex-sdk, sandbox, network, agent-runtime, debugging]
---

## Core Content

- The Codex SDK ThreadOptions field `networkAccessEnabled: true` does NOT, on its own, grant outbound network from inside the sandbox. The SDK translates that option into a scoped CLI config of the form `--config sandbox_workspace_write.network_access=true`. That config key only takes effect when the active sandbox profile is `workspace-write`. If the SDK is run with the default `read-only` sandbox, the scoped key is loaded but never consulted, and the agent run executes with no outbound network.
- The fix is mechanical: pass `sandboxMode: "workspace-write"` alongside `networkAccessEnabled: true`. The SDK then emits both `--sandbox workspace-write` and `--config sandbox_workspace_write.network_access=true`, and the two together actually open the network.
- Diagnostic signature when the option is set in isolation: any subprocess HTTP call inside the sandbox fails with `socket.gaierror: [Errno -3] Temporary failure in name resolution` or `urllib.error.URLError: <urlopen error [Errno -3] ...>`. DNS resolution itself fails, not just a specific endpoint, because the sandbox has no network namespace exit at all.
- `sandboxMode: "danger-full-access"` does open network as a side effect because it disables sandboxing wholesale, but it is a much broader fallback than the network-only case requires. Reach for `workspace-write` for network-only needs and reserve `danger-full-access` for cases where filesystem and process boundaries must also be lifted.

## Boundary Conditions

- Applies to the Codex SDK ThreadOptions surface (TypeScript / JavaScript). The exact CLI flag mapping is an SDK implementation detail and may change between SDK versions; the rule of thumb (network access is a property of the active profile, not a global) is the durable invariant.
- Does NOT apply when the SDK delegates to a binary on PATH that has its own profile selection — in that case the calling shell controls the sandbox.
- Does NOT apply if the workspace-write profile is already selected via another mechanism (CLI invocation env, parent profile inheritance, an outer wrapper that pins `--sandbox workspace-write`). The point of the heuristic is the SDK option in isolation, not the broader sandbox-config space.

## Limitations

- SDK-specific. Validated against the SDK version pinned in this build (~0.128.0). Future SDK refactors may merge the scoped configs into a single global `network_access` key, which would make the heuristic stale.
- The heuristic catches the option-in-isolation failure mode. It does not catch a separate class of network-failure where the sandbox profile is correct but the host itself has no outbound route (e.g. an air-gapped VM). When the host is unreachable, the same `gaierror -3` signature appears even with `workspace-write` selected.
- The mapping between SDK options and CLI flags is undocumented at the SDK README level in this version; reading the SDK's compiled `dist/index.js` was needed to confirm the translation. A future SDK may surface this in docs and make the heuristic less needed.

## Optional Notes

- Provenance:
  - ep.20260506T060000Z first surfaced `gaierror -3` from the embedded subagent when the SDK was configured with `networkAccessEnabled: true` but no `sandboxMode`.
  - ep.20260506T143000Z reproduced and characterized the failure mode as a sandbox profile mismatch rather than a credential or proxy issue.
  - ep.20260506T230500Z is the canonical Tier 1 fix episode: the resolution was a one-line ThreadOptions edit adding `sandboxMode: "workspace-write"`, after which the embedded subagent's HTTP calls completed end-to-end.
- Diagnostic shortcut: when an embedded SDK agent reports `gaierror -3` from a stdlib `urllib`/`requests` call, check the SDK invocation for an explicit `sandboxMode` BEFORE checking proxies, DNS resolvers, or container network policy. The SDK-option mismatch is the most common root cause and is the cheapest to verify.
- Cross-reference: this heuristic complements `kn.scaffold-adoption.heuristic.vendored-sdk-binary-system-upgrade-no-help` — both are reminders that the SDK's own configuration surface is the authoritative source of truth for sandbox-bound runtime behavior, not the surrounding system.
