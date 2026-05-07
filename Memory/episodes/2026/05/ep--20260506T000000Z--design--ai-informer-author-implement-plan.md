---
id: ep.20260506T000000Z.design.ai-informer-author-implement-plan
task_id: task.design.ai-informer-scaffold
domain: design
title: Author implement_plan.md gating artifact for AI Informer scaffolding
objective: Author /home/ubuntu/agents/builders/ai-informer-builder/implement_plan.md as the human-approved build manual that gates t2 subtasks 2-9, then send a Chinese no-markdown --await-reply mailbox message asking the human to approve before any scaffolding runs.
status: completed
eval_rounds: 1
last_edited_at: 2026-05-06
---

## Objective

Author `implement_plan.md` at workdir root — a ≤200-line build manual that references the build-a-codex-agent skill, calls out the basic-agent scaffold copy step, and pre-specifies the full post-scaffold directory layout — then mail the human (no markdown, Chinese, --await-reply) for approval to proceed with scaffolding.

## Context Snapshot

- Driver: human mail.20260505T162117Z.001 approved entering execution stage but conditioned it on first authoring an implement_plan.md with three explicit asks (reference build-a-codex-agent skill, copy basic-agent scaffold, pre-specify directory layout in detail).
- Prior episode: ep.20260505T154500Z.design.ai-informer-design-notifications-layer (PASS r1) — design tree is frozen.
- t2 has 9 subtasks; subtask 1 done, 2-9 pending. This episode authors a gating artifact that sits BEFORE subtask 2; no t2 subtask edits.
- Frozen design files (sources of truth, referenced by path in the plan): design/ai_informer_agent_design.md (≤100 lines), design/notifications.md (≤50 lines), design/skills/window-{08-urgency,12-opinion,18-deep}.md, design/subagents/{screener,researcher}.md.
- Already-authored config: config/window_{08,12,18}.json (sources.json shape, frozen).
- Build skill canonical path: .claude/skills/build-a-codex-agent/SKILL.md; helper at .claude/skills/build-a-codex-agent/scripts/copy.sh.
- Scaffold source resolves under harness engine, not the workdir: /home/ubuntu/agents/long-run-agent-harness/engine/scaffolds/basic-agent. Top level: AGENTS.md, CLAUDE.md, package.json, package-lock.json, tsconfig.json, src/, scripts/{run-once.sh, install-systemd-timer.sh, cron.example}, probes/.
- Hard caps: implement_plan.md ≤ 200 lines; webhook URL and id-fragment must NOT appear; builder-harness vocabulary (heartbeat/mailbox/episode/scheduled_tasks/todo_list) must not be applied to AI Informer's runtime/code-layout descriptions.
- Out of scope: copy.sh execution, scaffold drop, AGENTS.md authoring, .codex/agents/*.toml, .agents/skills/, ai_informer/ python package, config/notifications.json creation, cron install, knowledge note creation.

## Actions Taken

- Read SKILL.md and copy.sh; confirmed CLI shape `bash <skill>/scripts/copy.sh --dest <abs-path> [--name <slug>] [--no-install]`. Glob of `engine/scaffolds/basic-agent` returned: AGENTS.md, CLAUDE.md, package.json, package-lock.json, tsconfig.json, .gitignore, src/{entry/, loaders/, runtime/, trajectory/, CLAUDE.md}, scripts/{run-once.sh, install-systemd-timer.sh, cron.example, CLAUDE.md}, probes/{probe-stream.mjs, probe-skills.mjs}, .codex/agents/{web_researcher.toml.example, README.md} — matches planner's verified shape (with two extra CLAUDE.md helpers under src/ and scripts/, noted but not load-bearing).
- Authored `implement_plan.md` (164 lines) with sections Goal, Prerequisites and required reading, Starting point, Target directory layout, Build sequence (12 numbered steps), Verification strategy, Cron installation, Out of scope. Each numbered step carries input / action / done-when / risk.
- Updated `todo_list/202605/05.json`: t2 description now points at implement_plan.md as the contract for subtasks 2-9; subtask 6 rewritten to reference three window skills plus the shared helper (no "recommendation-rendering"). Subtask count unchanged (9); done-states unchanged (subtask 1 done; 2-9 false).
- Sent Chinese no-markdown `--await-reply` mailbox reply to human covering the six required points.

## Key Evidence

- `wc -l implement_plan.md` → 164 (≤200 cap).
- `grep -c "open.feishu.cn" implement_plan.md` → 0; `grep -c "<FEISHU_TOKEN_REDACTED>" implement_plan.md` → 0.
- `grep -c "build-a-codex-agent" implement_plan.md` → 5; `grep -c "basic-agent" implement_plan.md` → 5; `grep -c "copy.sh" implement_plan.md` → 4; `grep -c "scaffolds" implement_plan.md` → 3.
- `grep -c "recommendation-rendering" implement_plan.md` → 0.
- `grep -i -E "heartbeat|mailbox|episode|scheduled_tasks|todo_list" implement_plan.md` → 0 matches (no builder-vocabulary leakage into AI Informer's runtime descriptions).
- Directory tree section enumerates: AGENTS.md, .codex/agents/screener.toml, .codex/agents/researcher.toml, .agents/skills/window-08-urgency/SKILL.md, .agents/skills/window-12-opinion/SKILL.md, .agents/skills/window-18-deep/SKILL.md, .agents/skills/_shared/scripts/strip_for_screener.py, ai_informer/notifications/cli.py, ai_informer/notifications/registry.py, ai_informer/notifications/channels/feishu.py, config/notifications.json, config/window_{08,12,18}.json, Runtime/runs/<window>/<YYYYMMDD-HHMM>/, scripts/cron.example.
- Build sequence has 12 numbered steps; each has done-when criterion.
- `todo_list/202605/05.json`: 7 items; t2 description contains "implement_plan.md"; t2 subtasks done-states `[(1,True),(2,False),(3,False),(4,False),(5,False),(6,False),(7,False),(8,False),(9,False)]`; subtask 6 text no longer contains "recommendation-rendering".
- mailbox/human.jsonl tail-1: outgoing reply from "AI Informer Builder" to "human", `await_reply: true`, Chinese, zero markdown markers.

## Outcome

PASS round 1. Evaluator independently verified each spot-check: implement_plan.md (164/200) authored at workdir root with the full eight-section structure and a complete 12-step build sequence where every step carries input / action / done-when / risk lines (48 = 4×12 fields, fully shaped); all hard token gates clean (open.feishu.cn=0, <FEISHU_TOKEN_REDACTED>=0, build-a-codex-agent=5, basic-agent=5, copy.sh=4, scaffolds=3, recommendation-rendering=0); builder-vocabulary scan returned zero matches in implement_plan.md; the directory tree enumerates exactly the three window skills (no recommendation-rendering), the shared helper at .agents/skills/_shared/scripts/strip_for_screener.py, the ai_informer/notifications/{cli.py, registry.py, channels/feishu.py} package at workdir root parallel to ai_sources/, config/notifications.json as single source of truth, the three pre-existing config/window_*.json files, the per-run Runtime/runs/<window>/<YYYYMMDD-HHMM>/ artifact root, and scripts/cron.example. Plan-only discipline preserved: workdir top level shows only the new implement_plan.md plus pre-existing dirs — no AGENTS.md, no .codex/, no .agents/, no ai_informer/, no config/notifications.json. todo_list/202605/05.json: t2 description now references implement_plan.md as contract for subtasks 2-9; subtask 6 wording updated to name the three window skills plus the shared helper, with "recommendation-rendering" purged; subtask 1 stays done; 2-9 stay false. Mailbox reply mail.20260505T162948Z.001 confirmed Chinese, no markdown markers, await_reply true, with the ai_informer/ root-placement decision explicitly flagged for human pushback.

## Reflection

Three calls worth recording: (1) The mid-handoff catch that the planner had carried `.agents/skills/recommendation-rendering/SKILL.md` into the directory-tree expectation — a stale reference from the obsolete pre-rewrite skill set — was the right intercept. Inserting a planner correction between Phase 1 and Phase 2 is preferable to letting the executor produce a tree that contradicts the frozen design. The ratchet for next time: when a planner brief lists file paths that map to skill names, the main agent should cross-check those names against the current design's skill set before forwarding. (2) Authoring a build manual at this granularity (12 steps × 4 fields each = 48 explicit fields) up front is heavier than the typical design doc but pays off twice — the gating discipline (smoke after step 1, end-to-end smoke at step 11 before cron) becomes inspectable, and the per-step done-when criteria turn t2 subtasks 2-9 from prose intentions into verifiable contracts. (3) Surfacing the ai_informer/ placement decision in the mailbox rather than choosing silently was the right move because it is the one cross-cutting layout decision that affects every Python import path downstream; better to get it wrong before code than after. The evaluator's two distillation candidates (a "verify-plan-doc" checklist skill and a "plan-deviation transparency + load-bearing-decision flagging" heuristic) are noted but stay below promotion threshold at one observation each.

## Follow-up Actions

- Did NOT mark t2 done — t2 stays pending; mail.20260505T162948Z.001 (await_reply true) gates the next heartbeat. Subtasks 2-9 (scaffolding) become live only on human approval of the plan AND the ai_informer/ root placement.
- No new top-level Todo. The await_reply flag is the gate; adding a separate "verify human approval before t2 subtask 2" Todo would duplicate it.
- Open thread for next heartbeat: when the human's reply arrives, expected first scaffolding action is implement_plan.md step 1 — `bash .claude/skills/build-a-codex-agent/scripts/copy.sh --dest <workdir>` — followed by the post-copy probe smoke gate before any AI Informer customization. If the human pushes back on the ai_informer/ placement, update implement_plan.md's directory-tree section + step 9 + step 10 wording before proceeding.
- Knowledge promotion: NONE this episode. The two evaluator-flagged distillation candidates (plan-doc verification checklist; plan-deviation-and-load-bearing-decision-flagging heuristic) are recorded only; promote on second occurrence.
- Tooling note: design files for the three window skills will be the content source for steps 6/7/8 of the build sequence; the shared helper script's exact stripped-line format is locked at `<title> | <sources> [<source_categories>]: <summary excerpt> | <published_at> | id=<record_id>` per the prior episode's reconciliation.
