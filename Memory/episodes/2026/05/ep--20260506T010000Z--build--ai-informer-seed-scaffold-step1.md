---
id: ep.20260506T010000Z.build.ai-informer-seed-scaffold-step1
task_id: task.build.ai-informer-scaffold
domain: build
title: Seed basic-agent scaffold into workdir (implement_plan.md step 1)
objective: Seed the basic-agent scaffold files into the AI Informer Builder workdir without overwriting Builder-owned files, run the post-copy probe smoke gate, and confirm the scaffold is alive.
status: failed
eval_rounds: 1
last_edited_at: 2026-05-06
---

## Objective

Land the basic-agent scaffold files at workdir root (AGENTS.md, package.json, package-lock.json, tsconfig.json, src/, scripts/, probes/, .gitignore, .codex/, dist/ after build, node_modules/ after install) without damaging Builder-owned files (CLAUDE.md, ai_sources/, design/, mailbox/, todo_list/, Memory/, Runtime/, config/, .claude/, .git/, implement_plan.md), then run probe-stream as the smoke gate and confirm exit code 0.

## Context Snapshot

- Driver: human mail.20260505T170054Z.001 — broad approval to begin concrete development per implement_plan.md, ending in a real Feishu broadcast for human verification. This episode advances ONLY step 1 of that plan.
- Prior episode: ep.20260506T000000Z.design.ai-informer-author-implement-plan (PASS r1) — implement_plan.md is the contract.
- Todo: t2 subtask 2 ("Run build-a-codex-agent copy.sh to seed basic-agent into the workdir"). Mark done only after probe smoke passes.
- Hard collision discovered during planning: `copy.sh` at lines 26-29 hard-fails if `--dest` already exists. The implement_plan.md step 1 invocation `--dest /home/ubuntu/agents/builders/ai-informer-builder` will therefore fail outright. The executor must use a staging-then-merge approach.
- Hard collision #2: scaffold ships its own `CLAUDE.md` (2501 bytes, basic-agent decision layer). Workdir already has Builder's `CLAUDE.md` (7933 bytes, heartbeat behavioral rules). Builder's CLAUDE.md MUST WIN — do not overwrite it. The scaffold's CLAUDE.md content is information already covered by the build-a-codex-agent SKILL.md and is NOT needed at workdir root for runtime function.
- Hard collision #3: implement_plan.md step 1 says `npm run probe:stream`. The scaffold's package.json defines NO such script. Probe must be invoked as `node probes/probe-stream.mjs` directly. The implement_plan.md text is wrong on this point; treat the actual probe path as authoritative.
- Codex auto-discovery requires AGENTS.md, `.agents/skills/`, `.codex/agents/` to live at workdir ROOT, not in a subdirectory. So the scaffold contents (minus CLAUDE.md) must be placed at workdir root, not nested.
- `npm install` is a real side effect (downloads `@openai/codex-sdk` and dev deps; creates `node_modules/`). Human approval covers this. Do NOT pass `--no-install` — the probe-stream script imports `@openai/codex-sdk` and will fail without install.
- Boundary: do NOT proceed to step 2 (AGENTS.md replacement, .codex/agents/ authoring, .agents/skills/ authoring, ai_informer/ package, config/notifications.json) in this heartbeat regardless of how quickly step 1 completes.

## Actions Taken

- Read SKILL.md and copy.sh; confirmed planner findings (lines 26-29 hard-fail on existing dest; line 51 runs npm install + npm run build). No deviations.
- Captured pre-copy CLAUDE.md sha256 = 0329068621ccf9ae7116252f0fc0e83a78b7bfef28434aadd19f4d2ca26ee122 (7933 bytes) and workdir top-level snapshot.
- Ran `bash .claude/skills/build-a-codex-agent/scripts/copy.sh --dest /tmp/ai-informer-staging --name ai-informer-builder`. SUCCESS: scaffold copied, package.json renamed, `npm install` added 6 packages with 0 vulnerabilities, `npm run build` produced dist/ (entry, loaders, runtime, trajectory).
- Ran staging probe `node probes/probe-stream.mjs` from /tmp/ai-informer-staging. **FAILED**: verdict "error", exit 1, error string `Codex Exec exited with code 1: Reading prompt from stdin...`.
- HALTED per stop-and-report condition: "Staging probe-stream fails or verdict ≠ pass → STOP, do not move files." Did NOT move scaffold contents into workdir, did NOT mark Todo done, did NOT send a "success" mailbox.
- Diagnosed root cause: scaffold's package.json pins `@openai/codex-sdk: ^0.121.0`; SDK 0.121.0 ships its own vendored Codex CLI binary at node_modules/@openai/codex-linux-x64/vendor/.../codex (codex-cli 0.121.0). When invoked with the SDK's originator override (CODEX_INTERNAL_ORIGINATOR_OVERRIDE=codex_sdk_ts), the API rejects the request with: `400 invalid_request_error — The 'gpt-5.5' model requires a newer version of Codex. Please upgrade to the latest app or CLI and try again.` The system-installed `codex-cli 0.128.0` is unaffected; running `codex exec --experimental-json` directly (or the SDK on a host that bundles a newer CLI) succeeds. Verified by reproducing the SDK spawn manually with the vendored binary and observing the identical 400 error.
- Verified Builder workdir untouched: post-staging sha256 matches pre-copy; workdir top-level identical to pre-copy.
- Cleaned up /tmp/ai-informer-staging.

## Key Evidence

- Pre and post sha256 of /home/ubuntu/agents/builders/ai-informer-builder/CLAUDE.md both = 0329068621ccf9ae7116252f0fc0e83a78b7bfef28434aadd19f4d2ca26ee122 — Builder file untouched.
- copy.sh stdout key lines: `[build-a-codex-agent] copied scaffold -> /tmp/ai-informer-staging`, `added 6 packages, and audited 7 packages in 6s`, `> tsc`, `[build-a-codex-agent] installed + built`.
- Staging probe-stream JSON output: `{"probe":"probe-stream","verdict":"error","error":"Error: Codex Exec exited with code 1: Reading prompt from stdin...\n",...}` — exit code 1.
- Vendored codex (SDK-bundled, codex-cli 0.121.0) returns API error: `The 'gpt-5.5' model requires a newer version of Codex. Please upgrade to the latest app or CLI and try again.` System codex-cli 0.128.0 succeeds.
- Scaffold package.json pins `@openai/codex-sdk: ^0.121.0`. ^0.121.0 resolves to 0.121.0 (no newer 0.121.x or 0.12x.x ahead of the SDK has been released that the lock allows; an updated SDK with a refreshed bundled CLI is required).
- Implication for step 1 unblocking: scaffold needs its `@openai/codex-sdk` dependency bumped to a version whose vendored CLI accepts current API model defaults (or the SDK invocation needs a `model` override to a model the 0.121.0 vendored CLI supports). Either is a scaffold-level patch outside this episode's scope.

## Outcome

FAILED — execution-blocking obstacle that cannot be worked around within the episode boundary. Evaluator round 1 verdict: PASS on executor judgment, with explicit recommendation that the episode itself be set to `status: failed` per the advanced-episode-flow protocol's "execution-blocking obstacle" branch. The done-when criterion (probe-stream verdict pass + scaffold landed in workdir) was NOT met. The cause is upstream: scaffold's pinned `@openai/codex-sdk@^0.121.0` ships a vendored `codex-cli 0.121.0` whose request is rejected by the API with 400 invalid_request_error "The 'gpt-5.5' model requires a newer version of Codex." Independent verification confirmed: (a) Builder's CLAUDE.md sha256 unchanged at 0329068621ccf9ae7116252f0fc0e83a78b7bfef28434aadd19f4d2ca26ee122; (b) workdir top-level identical pre/post (no scaffold artifacts present); (c) `/tmp/ai-informer-staging` removed; (d) engine scaffold's package.json mtime predates this episode (no silent edit); (e) probe-stream.mjs unmodified (no model override injected); (f) mailbox tail unchanged from prior episode (no false-success message sent); (g) todo_list shows t2 subtask 2 still done: false. The executor correctly halted at the planner-defined stop-and-report boundary rather than silently working around the blocker via scaffold-level changes.

## Reflection

Three calls worth recording: (1) The staging-then-move workaround for `copy.sh`'s hard-fail-on-existing-dest worked exactly as designed — copy.sh succeeded, npm install + tsc succeeded, scaffold artifacts produced cleanly in `/tmp/ai-informer-staging`. The blocker surfaced only at the smoke gate, which is the correct gate position: a pre-move smoke catches scaffold defects before they pollute the Builder workdir. Without this discipline, the workdir would now contain a half-broken scaffold and rolling back would be a manual scrub. (2) The executor's decision to NOT silently bump the SDK pin or inject a model override was the right call. Both are scaffold-level changes that affect any downstream consumer of `engine/scaffolds/basic-agent/`. Editing a shared upstream artifact in response to a downstream gate failure is a classic layering violation; the right move is to escalate to the scaffold's owner (the human) for explicit direction. (3) The diagnostic chain — observing the failure, identifying the SDK as the carrier, locating the vendored binary path, comparing system vs vendored CLI versions, isolating that the API rejection is per-request not per-environment — is exactly the kind of investigation that distinguishes a real blocker from a misconfiguration. The evaluator's heuristic candidate ("when a vendored binary is pinned via an SDK dependency, upgrading the system-installed counterpart does not help — check the SDK's binary-resolution path before assuming a system upgrade fixes a vendor-shipped CLI defect") is generalizable enough to promote to a knowledge note this heartbeat.

## Follow-up Actions

- Set this episode `status: failed`. Done.
- Created new top-level Todo t8 ("Unblock basic-agent scaffold SDK pin so probe-stream passes — gates t2 subtask 2") with four subtasks enumerating the human-decision step, the apply-unblocker step, the re-run-staging-flow step, and the mark-subtask-2-done step. Done.
- t2 subtask 2 stays `done: false`. NOT flipped — probe never passed.
- Sent Chinese no-markdown `--await-reply` mailbox to human describing the blocker, the diagnostic, and the three options (a) bump SDK pin in engine scaffold, (b) inject model override in probe, (c) defer for upstream SDK refresh. Awaiting human decision.
- Knowledge promotion: write a heuristic note `heuristic--vendored-sdk-binary--system-upgrade-no-help.md` capturing the SDK-binds-vendored-binary pattern and the pre-action check (inspect findCodexPath() or equivalent before assuming a system-level fix helps). Generalizes beyond this specific SDK.
- Distillation candidates noted but NOT promoted: factual note about `@openai/codex-sdk 0.121.0`-specific API rejection (too volatile; will likely be fixed upstream within weeks); a "staging-then-merge" skill candidate (one occurrence — promote on second occurrence).
- Scaffold inaccuracies in implement_plan.md step 1 (`--dest <workdir>` invocation that copy.sh refuses; `npm run probe:stream` reference that does not exist) — NOT patched this episode. The patch should land in the next build-step episode that re-attempts step 1 after the unblocker is applied.
- Open thread for next heartbeat: gated on human reply to the await-reply mailbox. If human chooses (a) or (b), the next heartbeat opens an "engine scaffold patch" episode (which writes outside this Builder workdir into /home/ubuntu/agents/long-run-agent-harness/engine/scaffolds/basic-agent/) before re-running the staging-then-move flow. If (c), defer indefinitely with a check-in cadence.

