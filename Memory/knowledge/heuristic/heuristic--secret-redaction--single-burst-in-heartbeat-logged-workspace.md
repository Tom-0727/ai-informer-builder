---
id: kn.secret-redaction.heuristic.single-burst-in-heartbeat-logged-workspace
kind: heuristic
summary: When redacting secrets in a workspace whose executor commands are themselves heartbeat-logged into a tracked file, bind secret patterns to shell variables and bracket sed→assert→git add→commit into one yield-free shell burst.
last_edited_at: 2026-05-07
status: active
tags: [secret-redaction, git, heartbeat-discipline, debugging]
---

## Core Content

- The non-obvious failure mode: an executor agent that runs `sed -i` to scrub a secret from a workspace file will, in the same workspace, have its own verification commands logged into a tool-use transcript file (here `Runtime/events.jsonl`). Any verification step that types the literal secret as part of a `grep`, `rg`, or shell argument leaks the secret right back into the transcript file. If that transcript file is itself a redaction target, redaction is effectively undone between the sed pass and the commit, because the heartbeat tool dispatcher logs the verification command into the file the moment the command runs.
- Mitigation pattern, in order:
  - Bind the secret pattern to a shell variable before any verification, so the literal form never appears in command transcripts. Build the variable with `printf` from non-secret components and a suffix, e.g. `PATTERN=$(printf '%s/%s' "$HOST" "$SUFFIX")`. Both the variable name and the components stay non-secret; the literal form lives only in process memory.
  - Order the sed sweep so the heartbeat-logged transcript file is the LAST file written. Earlier targets can leak briefly, but the final pass overwrites those leaks before any commit-stage verification runs.
  - Bracket the redaction-and-commit sequence into a single shell invocation with no yield to the heartbeat tool dispatcher between steps. The chain looks like `sed -i ... events.jsonl && python3 -c '...JSON validate...' && git add -A && git grep "$PATTERN" ; git commit -m ...`, all in one bash call. No intervening tool calls, no separate verification step that the dispatcher can log between the sed and the commit.
- Verification corollary — verify by literal form, not by abbreviated prefix. When asserting secret absence post-commit, run `git grep <full-literal-form>` against the staged tree. Do NOT abbreviate to a hostname-only or token-prefix-only match. Telemetry and unrelated logs frequently match abbreviated patterns (e.g. a hostname appears in dozens of unrelated places) and will create false alarms; the literal full-form match is the only assertion that distinguishes leaked secrets from unrelated structurally-similar strings. The full-literal `git grep` returning 0 against the staged tree is the only acceptable green signal, even when the working tree may show post-commit telemetry that re-acquired the secret pattern (that is expected and is a separate problem, not a redaction failure).
- Post-commit reality: the working tree will, almost immediately after the commit, re-acquire the secret pattern in `Runtime/events.jsonl` because the agent's own commit-stage tool calls are themselves logged. This is expected. The redaction guarantee is "no commit ever shipped the secret", not "the working tree is permanently scrubbed". The next commit episode will absorb a fresh sweep.

## Boundary Conditions

- Applies when all three hold: (a) the workspace is git-managed and the agent commits its own records; (b) the agent's executor commands are tool-use-logged into a workspace file; (c) that log file is itself a redaction target. Remove any of the three and the heuristic relaxes.
- Less relevant when the agent has no command-transcript logging, or when redaction targets explicitly exclude the transcript path.
- The heuristic is about a redaction sweep at commit time. It does not replace write-time redaction at the source (mailbox-send, tool-use-logger). For ongoing operations, write-time redaction is more durable; this heuristic addresses the first-pass scrub against historical records.

## Limitations

- Works for the canonical first-commit-against-historical-record scenario. For ongoing redaction in a long-running agent, the right fix is write-time redaction at the mailbox-send and tool-use-logger surfaces, where the secret never reaches the persisted record. Single-burst sweeps remain valuable as a safety net but are an accumulating-debt model.
- The single-burst chain is fragile to any tool that adds an implicit yield between commands (e.g. some shell wrappers that echo each command separately to a logger). Validate that the chosen shell invocation truly does not yield; one safe default is a single `bash -lc '...'` with `&&` chaining and no `set -x`.
- The shell-variable-bound pattern still lands in the executor's process memory and may be visible to peer processes with sufficient privilege. The heuristic addresses transcript-logging leakage, not memory-introspection threats.

## Optional Notes

- Provenance:
  - ep.20260507T011000Z is the canonical exemplar — the first end-to-end redact-commit-push pass that adopted this discipline after observing the leak-and-relog behavior in earlier dry runs.
  - ep.20260506T230500Z first discovered the leak via the same single-burst safety gate that this heuristic codifies; that episode's halt was the input signal that the discipline was needed.
- Concrete pattern in this build: the chat-platform webhook URL of the form `<host>/<api-prefix>/<bot-token>` is the secret class. Verification used `PATTERN=$(printf '%s/%s' "$HOST_AND_API_PREFIX" "$TOKEN_SUFFIX")` so the literal full URL never appeared as a shell argument or in the bash history file. The placeholders `<FEISHU_WEBHOOK_REDACTED>` and `<FEISHU_TOKEN_REDACTED>` were used in committed files where the secret had previously appeared, so the structural shape of the original record was preserved without the secret content.
- Cross-reference: pairs with `kn.agent-architecture.heuristic.long-running-agent-secrets-accumulate-in-records` — this heuristic addresses the redaction mechanics, the cross-referenced one addresses the architectural choice of whether to redact at all versus changing the channel.
