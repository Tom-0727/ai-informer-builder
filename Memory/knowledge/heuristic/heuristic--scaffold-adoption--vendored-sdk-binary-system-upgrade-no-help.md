---
id: kn.scaffold-adoption.heuristic.vendored-sdk-binary-system-upgrade-no-help
kind: heuristic
summary: When an SDK pins a vendored binary, upgrading the system-installed counterpart does not help — inspect the SDK's own binary-resolution function before assuming a system upgrade fixes a vendor-shipped CLI defect.
last_edited_at: 2026-05-06
status: active
tags: [scaffold-adoption, dependency-pinning, debugging, sdk, codex]
---

## Core Content

- When an SDK ships and binds to a bundled binary via its own path-resolution function (e.g. Node SDKs that resolve binaries through `findCodexPath()` / `findBinaryPath()` / `require.resolve()` against `node_modules/<scope>/<package>-<platform>-<arch>/...`), upgrading the system-installed counterpart does NOT change runtime behavior. The SDK never consults `PATH`.
- A failing smoke test against an SDK-driven binary should be diagnosed by (1) reading the SDK's binary-resolution code path, (2) printing or logging the resolved path, (3) invoking the resolved binary directly with the same env the SDK sets (e.g. SDK-injected originator overrides), and (4) comparing version strings between the resolved binary and any system installation.
- The unblock for a vendor-shipped CLI defect is one of: (a) bump the SDK pin to a version whose vendored binary is fixed; (b) inject an SDK-level override that bypasses the broken behavior (e.g. `model` param, request headers); (c) defer until the upstream SDK publishes a refresh. Editing the system-installed binary is NEVER an unblock.

## Boundary Conditions

- This applies whenever a SDK ships per-platform optional-dependency binary packages (npm pattern: `@scope/package-os-arch` resolved at install time). Common in Codex SDK, esbuild, swc, sharp, Playwright, Puppeteer.
- Does NOT apply when the SDK explicitly delegates to a binary on `PATH` (rare in npm tooling; more common in Python tooling that shells out to system tools).
- Does NOT apply if the SDK exposes a documented "use my system binary instead" env var or constructor option — verify this exists before concluding the vendored binary is the only path.

## Limitations

- Pin-bumping (option a) requires editing shared scaffold/engine artifacts; that is a layering concern and may need explicit human approval before it can ship.
- Override injection (option b) may mask the true defect, leaving the SDK in a fragile state that breaks again as soon as the override is forgotten.
- Some SDKs aggressively cache their resolved binary path; clearing `node_modules` may be needed after a pin bump.

## Optional Notes

- Concrete instance: `@openai/codex-sdk@^0.121.0` resolves to a vendored `codex-cli 0.121.0` at `node_modules/@openai/codex-linux-x64/vendor/x86_64-unknown-linux-musl/codex/codex` via `findCodexPath()`. The bundled CLI's request was rejected by the API with 400 invalid_request_error "The 'gpt-5.5' model requires a newer version of Codex" while a separately-installed `codex-cli 0.128.0` succeeded on the same `codex exec --experimental-json` invocation. The SDK never used the 0.128.0 system install. Captured during ep.20260506T010000Z.build.ai-informer-seed-scaffold-step1 (status: failed).
- Diagnostic command pattern: `node -e "import('@openai/codex-sdk').then(m => console.log(m))"` to inspect the SDK module shape; then read its `dist/index.js` for the path-resolution function. For other SDKs, replace the import path.
- When proposing a pin bump to a human, include: current pinned version, latest stable version of the SDK, the API/server-side error that motivates the bump, and any known migration risks documented in the SDK changelog.
