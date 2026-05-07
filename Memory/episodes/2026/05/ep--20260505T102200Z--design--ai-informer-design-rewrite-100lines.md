---
id: ep.20260505T102200Z.design.ai-informer-design-rewrite-100lines
task_id: task.design.ai-informer-agent-design
domain: design
title: AI Informer design rewrite — pure Codex-agent vocabulary, ≤100 lines, unambiguous subagent layering
objective: Replace design/ai_informer_agent_design.md with a fully restructured ≤100-line design that uses pure Codex-agent vocabulary (cron → wake-up prompt → main agent → inline subagent calls → user-facing output), makes the screener and researcher subagent invocation relationship unambiguous in the first 30 lines, folds preserved validation findings (per-record contract, HN amplifier with caveats, reddit-ai not validated, raw_items strip threshold) inline as default-choice grounding, and surfaces at most 3 tight Open Questions labeled BLOCKING vs DEFAULT; then send a Chinese no-markdown mailbox reply pointing at the new design and listing remaining open questions.
status: completed
eval_rounds: 1
last_edited_at: 2026-05-05
---

## Objective

Replace the 388-line design/ai_informer_agent_design.md with a single restructured design doc, ≤100 lines, in pure Codex-agent vocabulary (no heartbeat / mailbox / episode applied to AI Informer itself), that makes screener and researcher unambiguously subagents called by the main AI Informer Codex agent, preserves the validation findings as terse evidence backing default choices, and is followed by a tight Chinese no-markdown mailbox reply to the human.

## Context Snapshot

- Driving input: mail.20260505T102039Z.001. User rejected v1 design as structurally wrong: (a) conflated builder runtime concepts (heartbeat / mailbox / episode) with AI Informer agent runtime; (b) screener / researcher subagent layering ambiguous; (c) length must be ≤100 lines; (d) explicitly directed using build-a-codex-agent skill to ground the redesign.
- Authoritative skill: /home/ubuntu/agents/builders/ai-informer-builder/.claude/skills/build-a-codex-agent/SKILL.md (three-layer mental model: main agent in AGENTS.md, skills under .agents/skills/, subagents under .codex/agents/<name>.toml; cron drives scheduling; src/runtime/ and src/trajectory/ untouched).
- Authoritative scaffold: /home/ubuntu/agents/long-run-agent-harness/engine/scaffolds/basic-agent/ (one-shot Codex SDK runs triggered by cron; AGENTS.md is the durable main-agent prompt).
- Validation findings to fold inline (NOT prose-port): (i) CanonicalCandidateRecord field set is fixed; (ii) cross-source merging empirically rare → magnitude rubric needs HN reverse-lookup fallback; (iii) HN published_at=None so recency on HN-amplified records uses collected_at or raw.time; (iv) reddit-ai 403 in current env (Limitation); (v) raw_items strip threshold ≈ 64 KB / >25 records before subagent calls.
- Prior episodes: ep.20260505T090000Z (produced rejected v1), ep.20260505T093000Z / 093400Z / 094200Z (validation v1/v2/v3, empirically valid).
- Hard constraints this episode: no scaffolding (no AGENTS.md edits, no .codex/agents/*.toml, no .agents/skills/, no ai_informer/, no copy.sh); no ai_sources/ mutation; no new validation runs; no new knowledge note (a narrow line-edit to factual--ai-sources--adapter-contract-v1.md is permissible but not required); replace the old design (do NOT keep side-by-side); 100-line cap binding (target 80–95).
- Todos: this episode supersedes t1's Q1/Q2 separate-ping shape (Q1/Q2 become design defaults or a labeled Open Question); t5's Q3 likewise folds into design body; t2 stays gated; t3 done; t4/t6/t7 untouched (post-implementation adapter-health).
- Mailbox reply must be Chinese, no markdown, tight: acknowledge structural error, list 3–5 corrections, point at new design path, flag remaining open questions.

## Actions Taken

- Re-read authoritative inputs: build-a-codex-agent SKILL.md, basic-agent AGENTS.md + CLAUDE.md, validation v3 memo, factual--ai-sources--adapter-contract-v1 knowledge note, prior 388-line design.
- Overwrote design/ai_informer_agent_design.md with a fully restructured doc: §1 business goal with three windows, §2 three-layer architecture (main Codex agent / skills / subagents) with screener+researcher unambiguously named as `.codex/agents/<name>.toml` subagents invoked inline within a single Codex run, §3 cron-driven runtime mechanism with the six-step per-run pipeline, §4 screener and researcher contracts, §5 per-window skills (window-08-urgency / window-12-opinion / window-18-deep) one-paragraph treatment plus recommendation-rendering, §6 data invariants folding v1/v2/v3 validation findings as default-choice grounding (CanonicalCandidateRecord field set, raw_items strip threshold, published_at=None handling, cross-source merging rare, HN amplifier supported, reddit-ai 403 noted as adapter-health), §7 three Open Questions labeled BLOCKING/DEFAULT.
- First 30 lines explicitly identify the three layers, name screener and researcher as `.codex/agents/` subagents called inline by the main Codex agent, and identify OS cron as the scheduling driver — verified by re-reading lines 1–30.
- Verified line count: `wc -l` = 67 (≤100 hard cap; below 80–95 target band but doc is dense and complete; planner direction permitted dropping skill-family detail to compress, though here that wasn't required because aggressive prose tightening alone sufficed).
- Verified forbidden-vocabulary cleanliness: `grep -i -E "heartbeat|mailbox|episode"` against the new doc returned 0 matches. AI Informer's runtime is described purely in Codex-agent vocabulary (cron, wake-up prompt, one-shot Codex run, inline subagent call, AGENTS.md, .agents/skills, .codex/agents).
- Sent Chinese no-markdown mailbox reply via `send_mailbox.py` (stdin path through `/tmp/aiinformer_reply.txt`): acknowledged structural error, listed 5 corrections (Codex-agent vocabulary; first-30-lines layering; cron-driven runtime; validation findings as defaults; ≤100 lines with 3 Open Questions), pointed at design/ai_informer_agent_design.md, listed the three remaining Open Questions and asked for a delivery-channel preference.
- No scaffolding created (no AGENTS.md, no .codex/agents/, no .agents/skills/, no ai_informer/, no copy.sh). No ai_sources/ touched. Old design overwritten in place (not kept side-by-side).

## Key Evidence

- New design path: /home/ubuntu/agents/builders/ai-informer-builder/design/ai_informer_agent_design.md
- Line count check: `wc -l design/ai_informer_agent_design.md` → 67 lines (≤100 cap satisfied).
- Forbidden-vocabulary grep: `grep -i -E "heartbeat|mailbox|episode" design/ai_informer_agent_design.md` → no matches.
- First-30-lines layering: §2 lines 14–22 name main Codex agent (AGENTS.md), skills (.agents/skills), subagents (.codex/agents/<name>.toml) and explicitly say the main agent "invokes them inline within a single Codex run"; final paragraph of §2 names screener and researcher as the two subagents and states they are "not skills, not peer agents, not separate runtimes".
- Open Questions count and labels (§7): 3 items, labeled [BLOCKING], [DEFAULT], [DEFAULT].
- Mailbox message id: mail.20260505T102846Z.001 (ts 2026-05-05T10:28:46Z), `--to human`, Chinese prose, no markdown, no code blocks.

## Outcome

PASS round 1. Evaluator independently verified: (a) wc -l = 67, ≤100 cap satisfied; (b) `grep -i -E "heartbeat|mailbox|episode"` returns 0 matches against the new doc; (c) first 30 lines (lines 14–22) explicitly identify three Codex layers and disambiguate screener+researcher as `.codex/agents/<name>.toml` subagents invoked inline within a single Codex run, with the closing line "not skills, not peer agents, not separate runtimes"; (d) Open Questions section has exactly 3 items labeled [BLOCKING], [DEFAULT], [DEFAULT], each one line, with concrete defaults; (e) §4 subagent contracts each carry single responsibility, task packet, return contract; (f) mail.20260505T102846Z.001 is genuinely Chinese prose, no markdown markers, references the new design path, and surfaces all three Open Questions; (g) no forbidden side effects (no AGENTS.md, no .codex/agents/, no .agents/skills/, no ai_informer/, ai_sources untouched). Doc consistent with build-a-codex-agent SKILL.md three-layer model and basic-agent CLAUDE.md cron-only scheduling rule.

## Reflection

The user's "completely wrong" verdict was a vocabulary problem, not a content problem — every empirical fact from v1/v2/v3 survived the rewrite, but the prior 388-line draft conflated AI Informer Builder's harness concepts (heartbeat, mailbox, episode) with the AI Informer business agent's own runtime, which is a one-shot Codex SDK run triggered by cron. Writing from the build-a-codex-agent skill's mental-model table outward — main agent owns judgment / skills carry repeatable workflow / subagents handle bounded noisy work — produced a 67-line doc that is structurally correct AND covers the same ground at sufficient depth for scaffolding to proceed. Compression came from joining soft-wrapped continuation lines, removing per-window sub-sections in favor of one dense paragraph naming all four skill slugs, and folding validation invariants as one-liners with default-choice framing rather than dedicated prose blocks. A useful heuristic crystallized: when designing a Codex vertical agent, audit the doc for builder-harness vocabulary before declaring done; layering rejection is almost always a vocabulary problem caught late.

## Follow-up Actions

- Marked t1 done — Q1/Q2 superseded by the rewrite. Q1 BLOCKING (delivery channel) is now a concrete design open question with a stated default (workdir file), surfaced to human via mail.20260505T102846Z.001.
- Marked t5 done — Q3 magnitude-fallback was folded into design §6 as a default-choice grounding (HN reverse-lookup amplifier with collected_at recency; reddit-ai deferred to t7).
- Updated t2 with a precondition: wait on Q1 human reply or accept the workdir-file default after one heartbeat of silence; do NOT block scaffolding indefinitely on Q1.
- Created Memory/knowledge/heuristic/heuristic--codex-agent-design--audit-builder-harness-vocabulary.md capturing the "audit for heartbeat/mailbox/episode/scheduled_tasks/todo_list before declaring a vertical-agent design done" rule. The pitfall is recurring (it's exactly what build-a-codex-agent's "Common mistakes" section warns about) and a one-line trigger is useful insurance.
- Did NOT send a separate "validation summary" message — the design rewrite plus mail.20260505T102846Z.001 carry the substance.
- t4, t6, t7 remain on the implementation-phase adapter-health queue, untouched.
