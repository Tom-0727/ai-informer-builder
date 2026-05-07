---
id: kn.agent-architecture.heuristic.long-running-agent-secrets-accumulate-in-records
kind: heuristic
summary: Secrets shared with a long-running agent through a logged channel propagate into durable agent records (mailbox, event log, episode bodies) and accumulate for the agent's lifetime; pick a handling strategy at design time, not at the first leak.
last_edited_at: 2026-05-07
status: active
tags: [agent-architecture, secret-handling, mailbox, runtime-records]
---

## Core Content

- The observed pattern: a single secret value handed to a long-running agent through one mailbox message does not stay in that one message. In this build, one webhook URL shared once via mailbox propagated into 15+ files spanning `Memory/episodes/`, `Runtime/events.jsonl`, `mailbox/human.jsonl`, and the pre-fill payloads of planner subagent contexts that were captured into the same logged surfaces. Each downstream surface that referenced the original mailbox content acquired its own copy of the secret, and the count grew over time as the agent's own tool-use transcripts re-quoted earlier turns.
- The architectural choice surface, framed as four mutually-non-exclusive options:
  - Option A — treat the agent's records as private-only with a rotation policy. Works if the records repo is provably private and the rotation cadence is shorter than the realistic compromise window. Cheap to adopt; depends on a strong privacy guarantee about the record store and an enforceable rotation discipline.
  - Option B — introduce write-time redaction at the source layer (mailbox-send hook, tool-use-logger hook). The secret never reaches the persisted record in the first place. Most durable. Requires engine-level changes at the points where messages and tool-use events are written; the redaction list must be kept in sync with the actual secret inventory.
  - Option C — batch-redact periodically before each commit. Lightweight to adopt; requires no engine change. Carries an accumulating-debt model: the working tree always re-acquires the pattern between commits, every commit episode pays a sweep tax, and the discipline can drift if a future commit episode forgets the sweep.
  - Option D — avoid sharing secrets through the logged channel entirely. Use out-of-band paths: drop the value into a local config file directly, fetch it from a secret manager at runtime, or hand it via an env var the agent reads but does not echo. Removes the leakage class structurally; requires that every flow that needed the secret be redesigned to not see it via mailbox.
- Decision frame: in deployments where the records are intentionally public-or-shared (the typical case for an agent whose memory and episodes form a published record of behavior), Options B and D are the only options that scale. Option C is acceptable as a first commit pass but is not a steady state. Option A is acceptable only when a strong privacy invariant on the records store can be guaranteed.

## Boundary Conditions

- Applies to any long-running agent that satisfies BOTH (a) accepts secrets through a logged channel (mailbox, tool-use transcripts, episode bodies) and (b) commits or publishes its own records to a public-or-shared destination. Both conditions are needed; either alone is insufficient to trigger the heuristic.
- Does NOT apply to ephemeral agents whose records are discarded at the end of a run.
- Does NOT apply to agents whose record store is provably private-by-construction (sealed local disk, end-to-end encrypted storage with no key escrow). In that case, secrets in records are not a leak, just a rotation question.
- Does NOT apply to one-shot Codex vertical agents triggered by cron — they are short-lived and their per-run state directories are typically gitignored. The heuristic targets the long-running builder-style agent, not the vertical agent it builds.

## Limitations

- The four options carry different operational weights and are not directly substitutable. Option B requires engine-level work; Option C is a procedural discipline that drifts; Option D may force a redesign of unrelated flows. The right answer is deployment-context-dependent.
- The heuristic is about handling at the channel level. It does not address upstream concerns like "should the human have shared the secret in the first place" — that is a separate operating-procedure question.
- The 15+ file count observed in this build is a property of this specific agent's logging surface. A leaner agent may show a smaller blast radius; a noisier one (e.g. one with verbose tool-use traces or multiple subagent layers) may show a larger one. The structural argument for picking a handling strategy at design time is unchanged.

## Optional Notes

- Provenance:
  - ep.20260506T143000Z first surfaced the safety-gate finding that a secret had landed in records; the gate halted further work pending an architectural decision.
  - ep.20260506T230500Z is the Stage D halt episode where the architectural-options framing was first articulated as the four-option surface above.
  - ep.20260507T011000Z is the redaction-execution episode that adopted Option C as the first-commit choice.
- This build picked Option C for the first commit (lightweight, no engine change required, single safety-gated sweep) but the recommendation for ongoing operations is to add Option B at the mailbox-send and tool-use-logger surfaces. Option C alone is not a steady state.
- The secret class in this build is the chat-platform webhook URL of the form `<host>/<api-prefix>/<bot-token>`. Committed files use the placeholders `<FEISHU_WEBHOOK_REDACTED>` and `<FEISHU_TOKEN_REDACTED>` to preserve the structural shape of the original record without the secret content. The literal forms are deliberately not present in this note.
- Cross-reference: pairs with `kn.secret-redaction.heuristic.single-burst-in-heartbeat-logged-workspace`, which addresses the mechanical redaction discipline once Option C has been selected.
