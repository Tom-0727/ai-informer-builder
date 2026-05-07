---
id: kn.ai-sources.factual.adapter-contract-v1
kind: factual
summary: Per-adapter empirical contract for ai_sources canonical records — published_at format, source_metrics shape, summary cleanliness.
last_edited_at: 2026-05-05
status: active
tags: [ai_sources, adapter, canonical-record, validation]
---

## Core Content

Empirical per-adapter contract for `ai_sources.core.collector.collect` records, observed against `--profile quick --limit 5` runs on 2026-05-05. Source: ep.20260505T093000Z.design.ai-informer-design-validation-v1 and Runtime/validation_*.json.

- `published_at` format
  - rss adapters (e.g. openai-blog, thegradient): RFC 822, e.g. `"Thu, 30 Apr 2026 00:00:00 GMT"`. Populated 5/5 in samples.
  - github_trending adapter: always `None` in samples (5/5). Recency dimension cannot rely on it; derive from collected_at or skip.
  - hacker_news / kr36 / reddit / product_hunt / html_index: not yet empirically sampled — assume RFC 822 or ISO 8601 until validated.

- `source_metrics` shape (lives on `Candidate`, propagated only via `raw_items[*].source_metrics`; not promoted onto `CanonicalCandidateRecord` directly)
  - rss adapters: `{}` (empty) in samples.
  - github_trending adapter: `{"stars": int}` populated for every record. NOTE: in 2026-05-05 sample all 5 stars=0 — likely adapter parser default; flag for follow-up before relying on the value.
  - hacker_news adapter (per source code, not yet empirically sampled): expected to expose score/comments via source_metrics.

- `summary` field cleanliness
  - openai-blog, thegradient: plaintext.
  - github_trending: contains raw HTML markup (`<p>`, `<div>`, `<a>`, `<img>`) in 3/5 samples. Downstream evidence-strength scoring or display rendering must clean HTML for github_trending records.

- `source_categories` taxonomy (always a list of strings on canonical records; first element is the underlying source's `category` from `source_profiles.json`)
  - openai-blog → `["official_lab"]`
  - thegradient → `["research_commentary"]`
  - github-trending → `["developer_signal"]`
  - These match the design's window→category mapping exactly.

- Per-record serialized size (full canonical dict, JSON-stringified, stripped means without `raw_items`)
  - openai-blog: ~1.4 KB / ~0.7 KB stripped
  - thegradient: ~1.6 KB / ~0.8 KB stripped
  - github-trending: ~2.6 KB / ~1.3 KB stripped
  - `raw_items` alone adds 600–1400 bytes per record. For a 50-record screener input, ~90 KB with `raw_items` vs ~46 KB stripped (worst-case ~128 KB).

- `merged_item_count` and `len(sources)` are 1 by construction in single-source runs. Empirically (validation v2, 2026-05-05) they were also 1 across two multi-source `--source-set` runs at `--profile quick --limit 3`: `official_labs` (9 sources, 24 records, `merged_duplicates=0`) and `editorial_and_research` (15 sources, 45 records, `merged_duplicates=0`). Reason: `dedupe_key` is canonicalized `url`, and distinct outlets rarely republish identical URLs. Cross-source amplification is therefore rare in practice for these source sets at this scale; designing rubric components on top of it requires either a larger `--limit`, a different source-set composition (e.g. official_labs + community-signal sources that link back to the labs), or a separate same-title/near-duplicate cross-source pass.

## Limitations

- Sample size is 5 records per source on a single calendar day. Not representative for low-volume sources or for unusual upstream content.
- hacker_news, kr36, reddit, product_hunt, html_index, rss-other have not been empirically sampled; their entries in this note are inferred from `core/sources/` code, not validated.
- The github_trending stars=0 anomaly is unconfirmed as a parser bug vs upstream HTML change. Treat the value with suspicion until ai_sources is allowed to be modified and a fix validated.

## Optional Notes

- Authoritative shape source: `ai_sources/core/models.py` (`Candidate`, `CanonicalCandidateRecord`) and `ai_sources/core/postprocess.py`.
- Validation memo: `design/ai_informer_design_validation_v1.md`.
- Triggering episode: ep.20260505T093000Z.design.ai-informer-design-validation-v1.
- Update this note when (a) a new adapter is empirically sampled, (b) the github_trending stars=0 anomaly is resolved, or (c) the rss/RFC-822 contract changes for any source.
