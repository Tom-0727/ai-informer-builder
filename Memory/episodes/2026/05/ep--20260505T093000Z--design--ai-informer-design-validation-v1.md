---
id: ep.20260505T093000Z.design.ai-informer-design-validation-v1
task_id: task.design.ai-informer-agent-design
domain: design
title: AI Informer design v1 — empirical validation against ai_sources.collect output
objective: Validate the load-bearing data assumptions in design §3 / §4 against a small live ai_sources.collect run and produce a concise validation memo whose findings either reinforce or revise specific design clauses.
status: completed
eval_rounds: 1
last_edited_at: 2026-05-05
---

## Objective

Run a bounded, representative ai_sources.collect against three sources (one per window) and produce design/ai_informer_design_validation_v1.md mapping each load-bearing data assumption from the design (CanonicalCandidateRecord field population, raw_items size, published_at format, source_categories taxonomy, window→source mapping) to an empirical observation and a concrete refinement decision; patch design/ai_informer_agent_design.md inline only for small/safe refinements.

## Context Snapshot

- Task continues from ep.20260505T090000Z.design.ai-informer-agent-design (PASS r1). That episode produced design/ai_informer_agent_design.md with two BLOCKING open questions sent to human (mail.20260505T090945Z.001). Those questions are unrelated to this episode's scope.
- Implementation (Todo t2) remains blocked by t1 awaiting human reply. Validation is design-phase work and is allowed.
- ai_sources.collect (`python -m ai_sources.core.cli` from this workdir) returns CanonicalCandidateRecord dicts via `records`, plus `source_counts`, `source_errors`, raw `items`. Key field set: record_id, dedupe_key, title, url, published_at, summary, tags, source_item_ids, sources, source_categories, merged_item_count, raw_items. raw_items[*] is the full original Candidate dict with `source_metrics`.
- Design §3.3 says main agent strips raw_items before passing to screener "if the record list is large" — needs concrete size evidence to set threshold.
- Design §4 window→source mapping uses categories: 08 → official_lab + cloud_platform + ai_hardware_platform + industry_news; 12 → editorial_practice + research_commentary + expert_analysis + newsletter; 18 → developer_signal + research_preprint + model_infra + agent_framework + developer_platform. Validate these category labels actually appear and are discriminating.
- Hard constraints: bounded HTTP (small --limit, e.g. 5 per source), no scaffolding, no ai_sources mutation, no mailbox-send, no skill or knowledge note creation, no AGENTS.md / *.toml.
- If a chosen source fails (network/parse), capture from `source_errors` and pick a backup; do not block on a single source.

## Actions Taken

- Read design doc §3.1–§3.4 and §4.1–§4.3, ai_sources/README.md, models.py, postprocess.py, source_profiles.json, collection_profiles.json (`quick`).
- Ran three single-source `ai_sources.core.cli` collections at `--profile quick --limit 5 --pretty` from the working directory:
  - `openai-blog` → `Runtime/validation_openai-blog.json` (5 records, source_errors empty).
  - `thegradient` → `Runtime/validation_thegradient.json` (5 records, source_errors empty).
  - `github-trending` → `Runtime/validation_github-trending.json` (5 records, source_errors empty).
- Computed per-source field population, published_at format, raw_items / record stringified sizes, source_metrics presence, and observed source_categories.
- Wrote `design/ai_informer_design_validation_v1.md` with run metadata, an 11-row validation table mapping each load-bearing assumption to an empirical observation and decision, a Patches Applied list, and an Open Questions section.
- Applied five small inline sentence-level patches to `design/ai_informer_agent_design.md`:
  - §3.3 strip rule made concrete (>25 records or >~64 KB).
  - §3.4 recency bullet adds RFC 822 format note and github-trending None caveat.
  - §3.4 magnitude bullet notes single-source caveat for `merged_item_count`/`sources`.
  - §3.4 evidence-strength bullet notes HTML-in-summary for github-trending.
  - §3.4 closing paragraph and §7.5 record empirical `source_metrics` population and the github-trending published_at caveat.
- Did not modify ai_sources/, did not create skills/knowledge notes/AGENTS.md/.codex/agents/.agents, did not send mailbox messages.

## Key Evidence

- Field population: every record (15/15) exposes the field set listed in §3.3. Validates the data-shape contract.
- `published_at`: openai-blog 5/5 RFC 822, thegradient 5/5 RFC 822, github-trending 0/5 (all None).
- `merged_item_count`: 1 for every record across all three runs (single-source runs cannot exercise cross-source amplification).
- Per-record serialized size: openai-blog ~1.4 KB / 0.7 KB stripped; thegradient ~1.6 KB / 0.8 KB stripped; github-trending ~2.6 KB / 1.3 KB stripped. raw_items[*] alone 600–1400 bytes/record. 50-record screener input estimate: ~90 KB with raw_items / ~46 KB stripped (heaviest case ~128 KB).
- `source_metrics`: empty for openai-blog and thegradient raw_items; populated as `{"stars": int}` for all 5 github-trending raw_items (first sample stars=0 → suspect parser default).
- `source_categories`: openai-blog `["official_lab"]`, thegradient `["research_commentary"]`, github-trending `["developer_signal"]`. All match the design's window→category mapping in §4.1, §4.2, §4.3 exactly.
- Artifacts: `Runtime/validation_openai-blog.json`, `Runtime/validation_thegradient.json`, `Runtime/validation_github-trending.json`, `design/ai_informer_design_validation_v1.md`.

## Outcome

PASS round 1. Evaluator independently re-verified all empirical claims by reading the three Runtime/validation_*.json artifacts (every record's field set, published_at format, source_metrics shape, source_categories match, per-record byte sizes) and confirmed the five inline patches to design/ai_informer_agent_design.md are sentence-level additions at lines 188, 212, 213, 215, 218, 386 — no contract boundary, rubric weight, schema, or section structure altered. No forbidden side effects (no AGENTS.md, no `.codex/agents/`, no `.agents/skills/`, no `ai_informer/`, no new mail past mail.20260505T090945Z.001, ai_sources untouched). Bounded-HTTP constraint honored.

## Reflection

The validation pass changed three design assumptions from "asserted" to "empirically grounded": (a) the §3.3 strip threshold is now concrete (>25 records or >~64 KB) instead of a vague "if large", (b) §3.4's recency dimension acknowledges RFC 822 (not ISO 8601) plus the github-trending None case so the screener and researcher will not silently mishandle missing timestamps, (c) §3.4's evidence-strength dimension flags HTML-in-summary for github-trending so the researcher's per-candidate prompt knows to clean it. Two findings are deferred as honest caveats: merged_item_count and len(sources) cannot be exercised by single-source runs (a multi-source follow-up is queued as a low-priority Todo), and the github-trending stars=0 anomaly across all 5 samples points at a possible adapter parser issue but ai_sources is read-only this episode (queued as a separate low-priority Todo for the implementation phase). The single load-bearing methodological note: when validating data-contract claims, single-source runs are insufficient for any signal that depends on cross-source merging.

## Follow-up Actions

- Updated Todos: kept t1 (await human) and t2 (scaffold, blocked by t1) as-is; added t3 (multi-source validation gap-filler — exercises merged_item_count and len(sources) > 1) and t4 (queue github-trending stars=0 investigation for the implementation-phase ai_sources fix round). Both t3 and t4 are low-priority and either gap-filler or post-unblock.
- Created knowledge note Memory/knowledge/factual/ai_sources_adapter_contract_v1.md capturing per-adapter published_at format, source_metrics shape, and summary cleanliness as observed; this fact set will be re-derived by every downstream design or implementation episode otherwise.
- Did NOT promote the "validation-memo pattern" to a skill yet (one occurrence so far per evaluator's note); revisit if the same shape recurs.
- Did NOT send any mailbox; the two BLOCKING items remain pending with the human and adding noise would not help.
