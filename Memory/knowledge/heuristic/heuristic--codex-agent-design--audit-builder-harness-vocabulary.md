---
id: kn.codex-agent-design.heuristic.audit-builder-harness-vocabulary
kind: heuristic
summary: Before declaring a Codex vertical-agent design done, grep it for builder-harness vocabulary — those words leaking in is the most common layering error.
last_edited_at: 2026-05-05
status: active
tags: [codex-agent, design, vocabulary, layering, build-a-codex-agent]
---

## Core Content

When designing a Codex-based vertical business agent (per `build-a-codex-agent` skill), audit the design doc for vocabulary that belongs only to the BUILDER's heartbeat-style harness, not to the vertical agent's runtime. Run a grep before declaring the design done:

```
grep -i -E "heartbeat|mailbox|episode|scheduled_tasks|todo_list" <design-doc>
```

Any non-trivial occurrence applied to the VERTICAL AGENT's runtime is a layering bug. Vertical Codex agents are one-shot @openai/codex-sdk runs triggered by cron. They do not have heartbeats, do not have a mailbox between the user and the agent at the runtime layer, and do not have episode files. Those are the BUILDER's harness concepts (the long-run-agent-harness wrapper that owns the builder's own work).

Correct vertical-agent vocabulary:
- "OS cron triggers a one-shot Codex SDK run" — not "heartbeat fires"
- "wake-up prompt for the active window" — not "wake-up message in mailbox"
- "main Codex agent invokes screener subagent inline within the run" — not "the agent posts a task to the screener"
- "user-facing output is written to <channel>" — not "the agent appends to mailbox"
- "cross-run state persists in workdir files" — not "episode records persist across heartbeats"

## When this heuristic fires

- Producing or revising a design doc for a Codex vertical agent.
- Reviewing someone else's vertical-agent design that was written by a builder agent (the builder's own vocabulary leaks in unconsciously).
- Triaging a "this design is wrong" rejection where the rejection cites layering or runtime mechanism.

## Limitations

- The heuristic is vocabulary-driven; it will not catch deeper structural errors (e.g. business judgment incorrectly placed in a subagent instead of the main agent's AGENTS.md, or a skill that is actually doing one-off task ownership).
- Mentions of those words may legitimately appear when the design discusses how the BUILDER agent will hand off work to the vertical agent — the rule is "applied to the vertical agent's own runtime," not "the literal word never appears."

## Optional Notes

- Triggering rejection: human feedback on 2026-05-05 (mailbox mail.20260505T102039Z.001) calling the prior 388-line design "completely wrong" because it conflated AI Informer Builder harness concepts with AI Informer's runtime.
- Resolution: ep.20260505T102200Z.design.ai-informer-design-rewrite-100lines (PASS r1) produced a 67-line replacement using pure Codex-agent vocabulary; grep returned 0 forbidden matches.
- Authoritative reference: `.claude/skills/build-a-codex-agent/SKILL.md` and `long-run-agent-harness/engine/scaffolds/basic-agent/CLAUDE.md` ("Use OS scheduling. Prefer cron on Linux. Do not add a scheduler daemon.").
