---
id: ep.20260505T093400Z.design.ai-informer-design-validation-v2
task_id: task.design.ai-informer-agent-design
domain: design
title: AI Informer design v2 — multi-source validation of cross-source signals
objective: Run a bounded multi-source ai_sources.collect against the official_labs source set (and optionally a second set) to empirically observe the cross-source signals merged_item_count and len(sources) that single-source v1 runs could not exercise, then write design/ai_informer_design_validation_v2.md as a tight memo and apply small inline patches to design/ai_informer_agent_design.md only when evidence directly supports a sentence-level refinement.
status: completed
eval_rounds: 1
last_edited_at: 2026-05-05
---

## Objective

Empirically determine whether cross-source amplification (merged_item_count > 1, len(sources) > 1) actually manifests in practice for the official_labs source set under quick/limit=3, record the finding in design/ai_informer_design_validation_v2.md, and either remove the single-source caveat in design §3.4 or sharpen it into a precise empirically-grounded statement.

## Context Snapshot

- Continues task.design.ai-informer-agent-design. Prior episode ep.20260505T093000Z.design.ai-informer-design-validation-v1 (PASS r1) established the per-record contract and added the §3.4 single-source caveat for merged_item_count / len(sources) that this episode is designed to retire or refine.
- Driving Todo: t3 (multi-source ai_sources.collect validation gap-filler). t1/t2 still blocked by human reply on two BLOCKING items; t4 deferred to implementation phase.
- official_labs source set (from ai_sources/core/config/source_sets.json) has 9 sources: aws-ml-blog, microsoft-ai-blog, meta-ai-blog, google-ai-blog, google-deepmind-blog, anthropic-news, nvidia-blog, mistral-news, openai-blog. Categories from source_profiles.json: 5 official_lab (meta, google-ai, google-deepmind, anthropic-news, mistral-news, openai-blog — actually 6 if openai counted), 2 cloud_platform (aws-ml, microsoft-ai), 1 ai_hardware_platform (nvidia). Mix of rss and html_index adapters; html_index ones (anthropic-news, meta-ai-blog, mistral-news) are the most likely failure points.
- Hard constraints (carried from v1): no scaffolding (no AGENTS.md, no .codex/agents/*.toml, no .agents/skills/, no ai_informer/), no ai_sources/ mutation, no mailbox-send (BLOCKING items still pending — do not pile on), no new skill creation, no new knowledge note unless multi-source data adds claims beyond factual--ai-sources--adapter-contract-v1. No rubric weight or section-structure changes to the design — only sentence-level patches.
- Companion knowledge note: Memory/knowledge/factual/factual--ai-sources--adapter-contract-v1.md already states "merged_item_count and len(sources) cannot be observed in single-source runs". This episode produces the data that lets that note be updated (or its limitation narrowed) — but only update the note if the evidence is durable.
- Validation memo v2 should be tighter than v1: do not redo the per-record contract table; focus only on cross-source signals (merged_item_count distribution, len(sources) distribution, source_categories on merged records, source_errors stability across html_index adapters).

## Actions Taken

1. Skimmed `ai_sources/core/postprocess.py` `_dedupe_key`: confirmed dedupe key is canonicalized `url`, falling back to `<source>:<title>` only when URL is missing. Records merge across sources only on identical canonical URL.
2. Ran `python -m ai_sources.core.cli --source-set official_labs --profile quick --limit 3 --output Runtime/validation_multi_official_labs.json --pretty`. Result: 24 records from 8 of 9 sources (meta-ai-blog 400'd), `merged_duplicates=0`, every record `merged_item_count=1` and `len(sources)=1`. Triggered the planner's "second run" branch.
3. Ran `python -m ai_sources.core.cli --source-set editorial_and_research --profile quick --limit 3 --output Runtime/validation_multi_editorial_and_research.json --pretty`. Result: 45 records, `source_errors={}`, `merged_duplicates=0`, every record `merged_item_count=1` and `len(sources)=1`.
4. Wrote `design/ai_informer_design_validation_v2.md` per the planner's tight format: header, run metadata, aggregate observations, cross-source-only validation table (4 rows), Patches Applied, Open Questions, Pointers. Did not redo the per-record contract from v1.
5. Applied three sentence-level inline patches to `design/ai_informer_agent_design.md`: §3.4 magnitude bullet (sharpened v1 caveat to a precise empirical statement), §3.4 evidence-strength bullet (one-sentence empirical observation appended for `len(sources)>1`), §7.5 (new bullet promoting "magnitude-signal fallback" as a deferred open question). No structural / rubric / schema changes.
6. Applied a narrow single-line edit to `Memory/knowledge/factual/factual--ai-sources--adapter-contract-v1.md` replacing the "cannot be observed in single-source runs" bullet with a multi-source-grounded statement (24+45 records, `merged_duplicates=0`). All other bullets unchanged.

## Key Evidence

- merged_item_count distribution across both runs (n=69): `{1: 69}`. Records with `merged_item_count > 1`: 0 / 69.
- len(sources) distribution across both runs (n=69): `{1: 69}`. Records with `len(sources) > 1`: 0 / 69.
- official_labs `source_counts`: `{aws-ml-blog:3, microsoft-ai-blog:3, meta-ai-blog:0, google-ai-blog:3, google-deepmind-blog:3, anthropic-news:3, nvidia-blog:3, mistral-news:3, openai-blog:3}`. `source_errors`: `{meta-ai-blog: "failed to fetch https://ai.meta.com/blog/: HTTP Error 400: Bad Request"}` (1/9 below the 5/9 stop-and-report threshold). Anthropic-news and mistral-news html_index adapters succeeded with `published_at` populated 3/3 each.
- editorial_and_research `source_counts`: 3 for all 15 sources. `source_errors={}`.
- Run outputs: `Runtime/validation_multi_official_labs.json`, `Runtime/validation_multi_editorial_and_research.json`.
- v2 memo: `design/ai_informer_design_validation_v2.md` (170 lines incl. blanks, tighter than v1's per-record table).
- Inline patches: `design/ai_informer_agent_design.md` lines ~213, ~215 (with v2 note appended) and §7.5 new bullet.
- Knowledge-note narrow edit: `Memory/knowledge/factual/factual--ai-sources--adapter-contract-v1.md` `merged_item_count` bullet only.

## Outcome

PASS round 1. Evaluator independently verified: (a) v2 memo present with the prescribed tight structure (cross-source-only, no per-record-contract redo); (b) Runtime artifacts shape and counts (24 + 45 records, {1: 69} distributions, merged_duplicates=0 on both, source_counts and source_errors as claimed); (c) three inline patches at lines 213, 215, 388 are sentence-level appends with no rubric weight or section-structure change (rubric line 232 confirmed unchanged); (d) dedupe_key mechanism claim cross-checked against ai_sources/core/postprocess.py `_dedupe_key` line 30; (e) knowledge-note edit is genuinely narrow (only the merged_item_count bullet); (f) HTTP cap honored (exactly 2 source-set runs, 0 retries); (g) no forbidden side effects (no AGENTS.md, no .codex/agents/, no .agents/skills/, no ai_informer/, mailbox tail still mail.20260505T090945Z.001, ai_sources/ untouched).

## Reflection

Empirical answer: cross-source amplification is essentially absent for these source sets at quick/limit=3 because ai_sources's `_dedupe_key` is canonical-URL-based, and distinct outlets rarely republish identical URLs. The v1 single-source caveat was the right hedge but understated the issue — magnitude as currently designed is delivered almost entirely by source_categories + title keywords + raw recency, not by cross-source amplification. This is a load-bearing assumption that quietly fails on real data; a deliberate amplifier (community-signal cross-check, larger limit, or near-duplicate title-cluster pass) would be needed to make magnitude meaningful as a cross-source signal. The fix is structural (rubric weight retune or fallback-signal addition) and therefore explicitly out of scope here; it is now §7.5's deferred decision and should bundle with the two existing BLOCKING items in the next human exchange. Methodologically, two heartbeats of validation (v1 single-source contract; v2 cross-source amplification) was the right cadence — each closed a specific empirical gap without overrunning the design phase.

## Follow-up Actions

- Marked t3 done (multi-source validation gap-filler — empirically resolved).
- Added t5 (BLOCKING design follow-up): bundle the magnitude-signal fallback decision (§7.5) into the next human message alongside the two existing BLOCKING items (delivery channel, rubric write-back). Three consolidated questions are friendlier than three separate pings.
- Added t6 (low-priority post-implementation): meta-ai-blog HTTP 400 adapter follow-up — out of scope while ai_sources is read-only; queue for the implementation phase.
- Did NOT send a new mailbox message this heartbeat — piling on the human while the original two BLOCKING questions are unanswered would be noise. The next human-bound message will include all three.
- Did NOT promote the cross-source dedupe finding to a separate knowledge note — already captured in the narrow edit to factual--ai-sources--adapter-contract-v1.md and design §3.4 / §7.5.
