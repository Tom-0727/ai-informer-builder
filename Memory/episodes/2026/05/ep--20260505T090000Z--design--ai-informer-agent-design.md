---
id: ep.20260505T090000Z.design.ai-informer-agent-design
task_id: task.design.ai-informer-agent-architecture
domain: design
title: AI Informer agent architecture design (design-only, pre-implementation)
objective: Produce a single approved design document at design/ai_informer_agent_design.md that fully specifies AI Informer as a Codex-app-server-based agent — covering business responsibility, AGENTS.md contract skeleton, time-window operating mechanism (08/12/18) over ai_sources.collect → screening subagent → per-candidate research subagent → main-agent synthesis, skill candidates per window, .codex/agents subagent definitions (lightweight screener and per-candidate researcher) with input/output contracts, side-effect boundaries, and any open questions — with NO scaffolding, NO AGENTS.md authoring, and NO subagent .toml writes this episode.
status: completed
eval_rounds: 1
last_edited_at: 2026-05-05
---

## Objective

Produce a single approved design document at `design/ai_informer_agent_design.md` that fully specifies AI Informer as a Codex-app-server-based agent — business responsibility, AGENTS.md contract skeleton, time-window operating mechanism (08/12/18) over `ai_sources.collect` → screening subagent → per-candidate research subagent → main-agent synthesis, skill candidates per window, `.codex/agents/` subagent definitions for the lightweight screener and per-candidate researcher with input/output contracts, side-effect boundaries, and explicit open questions — with no implementation, scaffolding, or AGENTS.md/subagent .toml authoring this heartbeat.

## Context Snapshot

- Hard precondition from human (mail.20260505T085754Z.001): design must come BEFORE any implementation; AGENTS.md contract, subagent definitions, and operating mechanism must be clear before code/scaffolding.
- Reference skill: `.claude/skills/build-a-codex-agent/SKILL.md` — three-layer mental model (main agent / skill / subagent), AGENTS.md as stable contract, subagents only for noisy/bounded/parallelizable work, single-responsibility + compact return contract per subagent.
- Data contract is fixed: `ai_sources.core.collector.collect(...)` returns `{records, items, source_counts, source_errors, raw_item_count, normalized_record_count, merged_duplicates, ...}`. Downstream consumes `records: list[CanonicalCandidateRecord]` with fields `record_id, dedupe_key, title, url, published_at, summary, tags, source_item_ids, sources, source_categories, merged_item_count, raw_items`. Design must reference these fields, not invent new ones.
- Time-aware delivery is a goal-level requirement: 08:00 high-urgency time-sensitive (major model/product releases from OpenAI/Anthropic etc.); 12:00 opinionated/conceptual (blogs/essays/forward-looking); 18:00 deep technical/practitioner (GitHub Trending projects, research, tools).
- Workdir state today: only `ai_sources/`, `Memory/`, `Runtime/`, `mailbox/`, `CLAUDE.md`. No AGENTS.md, no `.codex/agents/`, no `.agents/skills/`, no `design/`, no `ai_informer/`. Genuine fresh design.
- Archive `/home/ubuntu/agents/archive/ai-informer-builder/` is sanity-check material only — its AGENTS.md is generic scaffold-level, its `.codex/agents/` contains only generic episode-flow subagents (planner/executor/evaluator), no business screener/researcher. Treat archive as inspiration for shape, not as a spec to copy.
- Output language: human-facing reply will be Chinese with no markdown (per CLAUDE.md). The design doc itself is an artifact — structured headings are fine; primary user-facing prose in Chinese; technical contract field names stay English.
- Per CLAUDE.md, this episode must end with the design doc written but NOT yet sent to human; main agent decides whether to send a concise mailbox summary after evaluator PASS.

## Actions Taken

- Read authoritative inputs: `build-a-codex-agent` SKILL, `ai_sources/README.md`, `ai_sources/core/models.py` (CanonicalCandidateRecord field set confirmed), and the three configs under `ai_sources/core/config/` (source_sets, collection_profiles, source_profiles) to ground window→source-set defaults in real source ids and categories.
- Created `design/` and wrote a single design artifact at `design/ai_informer_agent_design.md` with all seven required sections, in order, with substantive content.
- Grounded all data-contract references in the actual `CanonicalCandidateRecord` fields (record_id, dedupe_key, title, url, published_at, summary, tags, source_item_ids, sources, source_categories, merged_item_count, raw_items); did not invent fields. Flagged in Open Questions that engagement signals would require reading `raw_items[*].source_metrics` since `source_metrics` is not promoted onto the canonical record.
- Defined the heartbeat pipeline and explicit boundary contracts (A through E) plus a shared window-profile schema used by both subagents.
- Specified the two subagents (screener, researcher) at skeleton level only — single responsibility, trigger description, input packet, side-effect boundary, compact JSON return contract — without authoring any `.toml`.
- Decided per-window skill candidates (window-08-urgency, window-12-conceptual, window-18-practitioner) plus cross-cutting skills (recommendation-rendering, feedback-ingestion) and recorded what is explicitly NOT a skill.
- Labeled Open Questions as BLOCKING vs DEFAULT for the four required items (delivery channel, window→source-set mapping, rubric weights, feedback loop).
- Did NOT execute copy.sh, did NOT author AGENTS.md, did NOT create `.codex/agents/*.toml`, did NOT modify `ai_sources/`, did NOT send mailbox.

## Key Evidence

- Design doc: `design/ai_informer_agent_design.md` — sections 1 through 7 present and substantively populated.
- CanonicalCandidateRecord field-set verified at `ai_sources/core/models.py:27-39`; rubric mapping in section 3.4 of the design references only those fields.
- Source-set choices in section 4 (window SOPs) reference real source ids and categories from `ai_sources/core/config/source_sets.json` and `ai_sources/core/config/source_profiles.json`.
- Subagent return contracts (section 5) are strict-JSON shapes whose fields are derived from `CanonicalCandidateRecord` (`record_id`) plus the five-dimension rubric in section 2.4.
- Three-layer separation honored: business ownership stays main-agent (sections 1, 2, 6), repeatable workflow lives in skills (section 4), bounded noisy work lives in subagents (section 5). Subagents propose; main agent commits.

## Outcome

PASS round 1. Design artifact `design/ai_informer_agent_design.md` (387 lines, 7 sections in order) satisfies the user's hard precondition that design must come before implementation. Evaluator independently verified: (a) all required sections present, (b) every data-contract reference grounded in real `CanonicalCandidateRecord` fields, (c) source-set / category references all exist in `ai_sources/core/config/`, (d) three-layer separation (main agent owns judgment; skills carry repeatable workflow; subagents propose only) honored, (e) stop-and-report constraints all respected (no AGENTS.md, no `.codex/agents/`, no `.agents/skills/`, no `ai_sources` mutation, no mailbox-send). Two BLOCKING open questions remain for human resolution before implementation: 7.1 user-facing delivery channel and 7.4 rubric write-back shape.

## Reflection

The load-bearing design choice was "subagents propose, main agent commits": the screener returns shortlist + droplist, the researcher returns evidence + a recommend flag, but the main agent owns the final commit to the user. This keeps business judgment in `AGENTS.md` rather than dispersing it into noisy bounded workers, exactly matching `build-a-codex-agent`'s three-layer model. A second important choice was making `window_profile` the single conduit for time-of-day bias into otherwise time-blind subagents — that pattern may generalize to other periodic-digest agents but one occurrence is too few to crystallize as a shared skill yet. One residual technical risk: `raw_items` size at the screener boundary; the design specifies the main agent strips `raw_items` before passing to the screener, which should be the implementation default.

## Follow-up Actions

- Sent concise Chinese mailbox summary to human (no markdown) reporting design completion and asking for resolution of the two BLOCKING open questions (delivery channel; rubric write-back shape) before implementation begins.
- Created Todos for: (1) await human resolution of the two BLOCKING questions, (2) after approval, scaffold the agent (AGENTS.md from §2, `.codex/agents/screener.toml` and `researcher.toml` from §5, three window-SOP skills and two cross-cutting skills from §4).
- Did NOT yet schedule 08/12/18 wake-ups; that is implementation-phase work and waits until the agent is scaffolded.
