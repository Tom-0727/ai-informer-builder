---
id: ep.20260505T094200Z.design.ai-informer-design-validation-v3
task_id: task.design.ai-informer-agent-design
domain: design
title: AI Informer design v3 — empirical fitness of hacker-news + reddit-ai as community-signal amplifiers
objective: Sample the hacker-news and reddit-ai adapters under a small bounded run and decide, with evidence, whether the v2 §7.5 magnitude-fallback proposal ("community-signal cross-check via hacker-news URLs") is adapter-supported as written, needs tightening (e.g. recency must be derived from raw.time / created_utc since published_at is empty), or needs widening (e.g. include reddit-ai vote/comment counts as a parallel amplifier signal); record findings in a tight design/ai_informer_design_validation_v3.md memo and apply only sentence-level inline patches to design §7.5 if findings change the wording the agent will eventually send to the human as t5's Q3.
status: completed
eval_rounds: 1
last_edited_at: 2026-05-05
---

## Objective

Empirically determine whether `hacker-news` and `reddit-ai` adapters produce CanonicalCandidateRecord output that actually supports the v2 §7.5 magnitude-fallback proposal — specifically whether `source_metrics` carries a usable score/vote count (and what type), whether `published_at` is populated (recency feasibility), whether `summary` is clean, and what `source_categories` taxonomy each maps to — then write design/ai_informer_design_validation_v3.md as a tight focused memo and apply small inline patches to design §7.5 only if findings change the eventual t5 Q3 wording. Optionally narrow the knowledge-note Limitations bullet to remove `hacker_news` and `reddit` from the unsampled list.

## Context Snapshot

- Continues task.design.ai-informer-agent-design. Prior episode ep.20260505T093400Z.design.ai-informer-design-validation-v2 (PASS r1) added §7.5 magnitude-fallback bullet at design line 388 mentioning "community-signal cross-check via `hacker-news` URLs"; reddit is not currently named there.
- Driving Todo: t5 (preparation only — does NOT consume t5; t5 still must wait for human reply on Q1/Q2 before being executed). This episode strengthens t5's eventual Q3 proposal with empirical adapter evidence.
- Adapter source ids confirmed in ai_sources/core/config/source_profiles.json: `hacker-news` (category `community_signal`, adapter `hacker_news`) and `reddit-ai` (category `social_forum`, adapter `reddit`). Both are single-id sources with subfeed selection driven by collection_profiles.json options (`hacker_news_feeds=["top"]`, `reddit_subreddits=["LocalLLaMA","singularity","artificial"]` under `quick`).
- Code-level pre-reads (already done at planning time) of ai_sources/core/sources/hacker_news.py and reddit.py establish what to expect — but EMPIRICAL sampling is still required for the validation memo:
  - hn source_metrics shape: `{"hn_points": int, "hn_comments": int}`. published_at: passed as `None` (Unix epoch present in raw.time only). summary: when enrich=True, `_article_preview` fetches the external URL and truncates 700 chars; when external_url is empty or hn-internal, falls back to top-3 comment preview. tags: `[<feed>]`.
  - reddit source_metrics shape: `{"reddit_upvotes": int, "reddit_comments": int}`. published_at: never passed (defaults None — created_utc is in raw_data not propagated). summary: `selftext_html` / `selftext` cleaned and truncated to 700 — for link-style posts this will be empty.
  - quick profile sets `enrich=true`. limit=5 on hacker-news means up to 5 external article fetches plus possibly 3 comment-item fetches per fallback — bounded but not zero. limit=5 on reddit-ai means 1 hot.json fetch per subreddit (3 subreddits) — small.
- Hard constraints carried forward from v2: NO scaffolding (no AGENTS.md, no .codex/agents/*.toml, no .agents/skills/, no ai_informer/), NO ai_sources/ mutation, NO mailbox-send, NO new skills, NO new knowledge note (only narrow line-edit allowed to existing note), NO rubric weight change, NO §-structure change. Sentence-level inline patches only. Bounded HTTP — at most 2 cli runs (one per source), `--profile quick --limit 5`.
- If a source id is somehow missing or the run fails persistently after 1 retry, capture the failure as a finding and stop; do not invent ids and do not expand HTTP.

## Actions Taken

- Ran `python -m ai_sources.core.cli --source hacker-news --profile quick --limit 5 --output Runtime/validation_v3_hacker_news.json --pretty`. Success: 5/5 records, `source_errors={}`.
- Ran `python -m ai_sources.core.cli --source reddit-ai --profile quick --limit 5 --output Runtime/validation_v3_reddit_ai.json --pretty`. Failure: 0/5 records, HTTP 403 Blocked on `https://www.reddit.com/r/LocalLLaMA/hot.json`. Per stop-and-report rule, did not retry (would require ai_sources UA mutation).
- Wrote `design/ai_informer_design_validation_v3.md` (tighter than v2, two adapters only) with per-adapter contract, fitness verdicts (HN suitable-with-caveats; reddit-ai not-suitable in current env), patches-applied diff, and open questions.
- Applied 1 sentence-level inline patch to `design/ai_informer_agent_design.md` §7.5 (line 388) — tightened the v2 community-signal-amplifier sentence to acknowledge HN canonical `published_at=None` and to explicitly NOT widen to reddit-ai (since reddit-ai returned 0/5).
- Did NOT apply the optional knowledge-note line-edit to `factual--ai-sources--adapter-contract-v1.md` line 45. Gate was "both adapters sampled cleanly"; reddit-ai was not. Reasoning recorded in v3 memo.

## Key Evidence

- `Runtime/validation_v3_hacker_news.json`: 5 records, `source_categories=["community_signal"]` 5/5, `source_metrics={"hn_points","hn_comments"}` 5/5 with hn_points min/median/max=100/123/502 and hn_comments min/median/max=6/13/74, `published_at=None` 5/5, `summary` plaintext (700–848 chars, no HTML tags), canonical `url` is the external article URL, HN discussion URL preserved at `raw_items[0].raw.hn_url`, Unix epoch at `raw_items[0].raw.time`. AI-relevance of `top` frontpage sample: ~1–2 of 5.
- `Runtime/validation_v3_reddit_ai.json`: `item_count=0`, `source_errors={"reddit-ai": "failed to fetch https://www.reddit.com/r/LocalLLaMA/hot.json?limit=10&raw_json=1: HTTP Error 403: Blocked"}`. Documents the failure mode for the implementation-phase adapter-health queue.
- Design patch confirmed at `design/ai_informer_agent_design.md` §7.5 line 388 (single-sentence rewrite within the existing v2 bullet; before/after diff captured in v3 memo Patches Applied).
- Open Questions promoted: reddit-ai 403 alongside t4 github_trending stars=0 in the implementation-phase adapter-health queue; HN AI-relevance density (favor reverse lookup); HN `published_at=None` knock-on for the 18-window recency gate.

## Outcome

PASS round 1. Evaluator independently re-verified: (a) v3 memo present with the prescribed tight structure (8200 bytes vs v2's 10015, scope genuinely narrower); (b) Runtime/validation_v3_hacker_news.json carries 5/5 records with non-trivial hn_points/hn_comments distributions (100–502 / 6–74), published_at=None 5/5, summary plaintext, source_categories=["community_signal"]; (c) Runtime/validation_v3_reddit_ai.json documents a hard 403 in source_errors with item_count=0 and no retry; (d) the single inline patch at design §7.5 line 388 is a sentence-level rewrite embedded in the existing v2 bullet — no new bullet, no rubric weight change, no §-structure change; (e) the knowledge-note Limitations bullet at line 45 is correctly left intact because the "both adapters sampled cleanly" gate was violated; (f) HTTP cap honored (2 cli runs, 0 retries past the 1 budget); (g) no forbidden side effects (no AGENTS.md, no .codex/agents/, no .agents/skills/, no ai_informer/, mailbox tail mail.20260505T090945Z.001, ai_sources untouched).

## Reflection

The empirical answer for t5's Q3: hacker-news as a community-signal amplifier is suitable-with-caveats — its source_metrics carry meaningful hn_points and hn_comments distributions, but canonical published_at is None so recency on HN-amplified records cannot use the canonical field; recency must come from collected_at, raw_items[0].raw.time, or a future adapter change. AI-relevance density on the HN top frontpage is sparse (~1–2 of 5 records), which suggests the right shape for the amplifier is reverse lookup (for each AI-source URL, query HN to look it up) rather than forward iteration of HN top stories. reddit-ai cannot currently be validated in this environment due to a UA-based 403; this is now durable adapter-health work alongside t4 (github-trending stars=0) and t6 (meta-ai-blog 400). The executor's restraint on the knowledge-note edit (gate said "both clean", reddit failed, so no edit) was the right call — half-edits to a Limitations list rot quickly. Methodologically, three validation rounds in a row each closed a distinct empirical gap (per-record contract; cross-source amplification; community-signal adapter contract); v3 should be the last validation round before t5's Q3 lands with the human, and the evaluator's "skill candidate (mild)" — packaging the sample-and-summarize-adapter pattern into a small script — is worth flagging if a fourth round occurs but not creating preemptively.

## Follow-up Actions

- Refined t5: Q3's eventual wording will now ground the proposal in v3 evidence — hacker-news (suitable-with-caveats, recency-via-collected_at, reverse-lookup shape) as the primary community-signal amplifier; reddit-ai noted but explicitly NOT recommended as parallel amplifier until the 403 is resolved.
- Added t7: reddit-ai HTTP 403 adapter-health follow-up (post-implementation phase, joins t4 and t6 on the adapter-health queue). Covers UA fix in ai_sources/core/sources/reddit.py and re-validation.
- Did NOT send a new mailbox message — t5 still requires human reply on Q1/Q2 first; piling on while pending is noise. Q3 wording is now strengthened for whenever t5 fires.
- Did NOT promote the validation-memo pattern to a skill — three uses is the threshold, but if no fourth round occurs (likely), the cost of authoring the skill exceeds the benefit. Defer until a clear fourth occurrence.
- Did NOT update the knowledge note this heartbeat (gate not met). When the reddit adapter-health work in t7 lands, both bullets (Limitations line 45 and Core Content adapter-by-adapter section) can be updated together.
