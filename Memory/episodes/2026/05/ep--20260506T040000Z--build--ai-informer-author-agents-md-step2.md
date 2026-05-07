---
id: ep.20260506T040000Z.build.ai-informer-author-agents-md-step2
task_id: task.build.ai-informer-scaffold
domain: build
title: Author AI Informer AGENTS.md (implement_plan step 2)
objective: Replace the scaffold's 1010-byte stock AGENTS.md with an AI-Informer-specific Codex system prompt (≤150 lines) that encodes identity, three-layer model, four-stage runtime, subagent invocation policy, time-window assignments, output format, notification CLI policy, invariants, and out-of-scope — referencing design files by path-and-section rather than re-spelling them — then mark t2 subtask 3 done and send a Chinese no-markdown progress mailbox.
status: completed
eval_rounds: 1
last_edited_at: 2026-05-06
---

## Objective

Replace the scaffold's 1010-byte stock AGENTS.md at workdir root with an AI-Informer-specific Codex system prompt (≤150 lines) that paraphrases business identity, encodes the three-layer model and four-stage runtime, names the subagent invocation policy and time-window-to-skill assignments, fixes the output format and notification CLI invocation, lists invariants, and points at design files by path-and-section rather than duplicating their content. Then flip todo_list/202605/05.json t2 subtask 3 to done and send a Chinese no-markdown progress mailbox.

## Context Snapshot

- Prior episode: ep.20260506T030000Z.build.ai-informer-seed-scaffold-step1-retry (PASSED r1) — scaffold landed at workdir root, builder CLAUDE.md sha256 0329068621ccf9ae7116252f0fc0e83a78b7bfef28434aadd19f4d2ca26ee122 preserved.
- Implement_plan step 2 is the only step in scope this heartbeat. Steps 3-12 stay deferred.
- Content authorities (validated in planning): design/ai_informer_agent_design.md (63 lines, §1–§6), design/notifications.md (49 lines), design/skills/window-{08-urgency,12-opinion,18-deep}.md (34–35 lines each), design/subagents/{screener,researcher}.md (32–35 lines).
- Stock AGENTS.md at workdir root is the generic basic-agent placeholder (12 lines, 1010 bytes). It has no load-bearing Codex SDK invocation guidance to preserve — the build-a-codex-agent SKILL.md is the authoritative scaffolding doc, not the placeholder.
- Hard cap: AGENTS.md ≤150 lines.
- Forbidden tokens (Builder-harness vocabulary, must not appear in AGENTS.md as AI Informer runtime concepts): heartbeat, scheduled_tasks, todo_list, mailbox, episode. (mailbox and episode are allowed only if they unambiguously refer to the Builder context, but in practice should not appear at all in AGENTS.md.)
- Forbidden literals: the Feishu webhook URL (<FEISHU_WEBHOOK_REDACTED>) — webhook URLs live only in config/notifications.json, which is step 10 work and not authored this heartbeat.
- Builder CLAUDE.md is BUILDER behavioral rules (this agent's), unrelated to the AI Informer's runtime. Not to be confused with AGENTS.md (AI Informer's authoritative system prompt for the Codex SDK). Per implement_plan step 2 there is also the question of whether to retain/adjust/remove the SCAFFOLD's CLAUDE.md (Claude-Code convention, not authoritative for Codex). Scaffold CLAUDE.md under workdir root is the basic-agent's, distinct from Builder CLAUDE.md.
- AI Informer is one-shot Codex SDK per OS cron entry — not a heartbeat-style long-running agent. AGENTS.md voice should reflect that.
- Per-run intermediate-artifacts dir convention: Runtime/runs/<window>/<YYYYMMDD-HHMM>/ — per design §3 and §6.
- Wake-up shape: "现在是 HH:00，请执行 <window 主题> 推荐" — per design §3.
- Notification CLI invocation: `python -m ai_informer.notifications.cli --message-file <output.md path>` (default broadcast) / `--channel <name>` (single channel) — per design/notifications.md.
- Time-window-to-skill mapping: 08:00 → window-08-urgency (urgency); 12:00 → window-12-opinion (conceptual); 18:00 → window-18-deep (practitioner).
- Output format: Chinese no-markdown; ≤200 字 summary + per-item Title / Why-worth-reading / URL; up to 15 items; quality over volume — per design §1.
- Out-of-scope to call out in AGENTS.md: cron installation, ai_sources adapter mutation, anything requiring human authorization.
- Boundary: AGENTS.md authored + t2 subtask 3 flipped + progress mailbox sent. Do not proceed to step 3 (screener.toml).

## Actions Taken

- Read .claude/skills/build-a-codex-agent/SKILL.md (AGENTS.md guidance lines 62-72), design/ai_informer_agent_design.md (63 lines, §1-§6), design/notifications.md (49 lines), design/skills/window-{08-urgency,12-opinion,18-deep}.md, design/subagents/{screener,researcher}.md.
- Verified preconditions: workdir AGENTS.md was the 1010-byte stock placeholder; Builder CLAUDE.md sha256 = 0329068621ccf9ae7116252f0fc0e83a78b7bfef28434aadd19f4d2ca26ee122; no scaffold CLAUDE.md present at workdir root (only Builder's 7933-byte CLAUDE.md).
- Authored /home/ubuntu/agents/builders/ai-informer-builder/AGENTS.md overwriting the stock placeholder. Final shape: 150 lines, 7394 bytes, nine sections in the prescribed order — Identity / Operating model / Runtime mechanism (four-stage flow with per-run dir convention) / Subagent invocation policy / Skills assignment / Output format / Notification delivery / Invariants / Out of scope.
- Cited design files by path-and-section for every load-bearing section; out-of-scope authored as fresh framing (no §7 citation, since the design has none).
- Flipped todo_list/202605/05.json t2 subtask 3 to done via Edit; verified all other subtask states unchanged (t2 sub 1-2 still done, sub 4-9 still false; t2.done still false).
- Sent Chinese no-markdown progress mailbox mail.20260506T030348Z.001 to human (no --await-reply) summarizing what changed, line/byte counts, design-file references, verification gates passed, and next step.

## Key Evidence

- AGENTS.md verification (run after final edit): line count 150 (at cap), byte count 7394 (was 1010, delta +6384), grep -c "open.feishu.cn" = 0, grep -c "<FEISHU_TOKEN_REDACTED>" = 0, grep -i -c -E "heartbeat|scheduled_tasks|todo_list" = 0, grep -i -c -E "mailbox|episode" = 0.
- Builder CLAUDE.md sha256 unchanged across run: 0329068621ccf9ae7116252f0fc0e83a78b7bfef28434aadd19f4d2ca26ee122.
- All seven design files referenced verbatim in AGENTS.md (verified by `grep -oE "design/(ai_informer_agent_design|notifications|skills/window-08-urgency|skills/window-12-opinion|skills/window-18-deep|subagents/screener|subagents/researcher)\.md" | sort -u`): design/ai_informer_agent_design.md, design/notifications.md, design/skills/window-{08-urgency,12-opinion,18-deep}.md, design/subagents/{screener,researcher}.md. Section anchors used: §1, §2, §3, §4, §5, §6.
- AGENTS.md contains all three time windows (08:00, 12:00, 18:00), the four numbered stages (Collect / Strip + screen / Research / Deliver), the per-run dir literal `Runtime/runs/<window>/<YYYYMMDD-HHMM>/`, and the notifications CLI invocation literal `python -m ai_informer.notifications.cli --message-file`.
- todo_list/202605/05.json post-edit: t2.done=False (correctly preserved); t2 subtasks: sub1 done, sub2 done, sub3 done (newly flipped), sub4-9 still false. All other top-level Todos (t1,t3,t4,t5,t6,t7,t8) untouched.
- Mailbox tail-1 (mailbox/human.jsonl): id=mail.20260506T030348Z.001, to=human, from=AI Informer Builder, await_reply=False; body is Chinese, no markdown markers (no #, no **, no backticks, no triple-backticks, no leading "- "). Body covers the six required points (file written, section structure, line/byte counts, design-file references, verification gates, next-step pointer to step 3 screener.toml).
- No other workdir top-level files modified outside AGENTS.md, todo_list/202605/05.json, mailbox/human.jsonl (append), and this episode file.

## Outcome

PASS round 1. Evaluator independently verified eighteen spot-checks: AGENTS.md contains all nine prescribed sections (Identity, Operating model, Runtime mechanism, Subagent invocation policy, Skills assignment, Output format, Notification delivery, Invariants, Out of scope) at 150 lines (exactly at cap, content-dense not padded) and 7394 bytes (vs. 1010-byte stock placeholder); voice is Codex system prompt (declarative second-person, judicious bullets, no markdown noise); zero matches on every forbidden-token gate (open.feishu.cn, <FEISHU_TOKEN_REDACTED>, heartbeat/scheduled_tasks/todo_list, mailbox/episode); Builder CLAUDE.md sha256 unchanged at the canonical baseline; all three time windows present with prioritization rationale and skill mapping; four-stage flow with per-run dir literal `Runtime/runs/<window>/<YYYYMMDD-HHMM>/` cited twice; notifications CLI literal `python -m ai_informer.notifications.cli --message-file <run-dir>/output.md` present; wake-up message shape `现在是 HH:00，请执行 <window 主题> 推荐` present; all seven design-file references present (main design + notifications + three window skills + two subagents) with citations correctly bounded to existing §1-§6 (no fabricated §7); todo_list edit confined to t2 subtask 3 flip; mailbox tail confirms outgoing Chinese no-markdown progress with await_reply false; .codex/agents/ contains only scaffold's stock README.md + web_researcher.toml.example confirming the step-3 boundary was respected. Two distillation candidates noted (a "Codex agent artifact authoring contract" skill candidate; a "verify section exists before citing" heuristic candidate) — both at low signal, recorded only.

## Reflection

Three calls worth recording: (1) The reference-not-duplicate discipline made the line cap workable. AGENTS.md owns identity + routing + the runtime contract; per-window relevance criteria, screener decision rules, researcher web bounds, and channel adapter shapes stay in their own design files and are pointed at by path-and-section. Without this discipline AGENTS.md would have absorbed several hundred lines of design content and become a parallel source of truth. (2) The planner's pre-validation that the design has only §1-§6 (no §7) prevented a citation-to-nonexistent-section bug at authoring time. Authoring "Out of scope" as fresh framing rather than reaching for a "design §7 says..." reflex is the kind of small correction that compounds across an agent's lifetime — the scaffold-style framing trap is real. (3) Hitting the 150-line cap exactly is a deliberate trade-off — the executor declined to drop a design-file pointer or compress a load-bearing invariant in order to undercut the cap. This is correct restraint: the cap exists to prevent parallel-source-of-truth drift, not to force arbitrary compression of necessary content. The cap held; the content held. Future heartbeats should treat 150 as a hard ceiling for AGENTS.md and reach for trim-by-pointer if it ever pushes against it.

## Follow-up Actions

- Marked t2 subtask 3 done — already applied by executor; evaluator verified.
- t2 stays pending; subtasks 4-9 remain false. Step-3 boundary (no .codex/agents/screener.toml authoring) was respected — verified by evaluator.
- Open thread for next heartbeat: t2 subtask 4 = step 3 of implement_plan.md = author .codex/agents/screener.toml from design/subagents/screener.md. Subsequent heartbeats: subtasks 5-9 in order.
- Knowledge promotion: NONE this episode. The two distillation candidates from the evaluator (skill candidate "Codex agent artifact authoring contract"; heuristic candidate "verify section exists before citing") are below promotion thresholds at this signal level. Watch for recurrence in step 3 (.toml authoring) and step 4 (.toml authoring) — if a similar reference-not-duplicate + line-cap + section-citation discipline holds across both, that would justify promoting the skill candidate.
- The pre-existing heuristic at Memory/knowledge/heuristic/heuristic--codex-agent-design--audit-builder-harness-vocabulary.md remains the durable record of the vocabulary-layering discipline that this episode also exercised (zero forbidden-token matches across builder-harness terms in AGENTS.md).
