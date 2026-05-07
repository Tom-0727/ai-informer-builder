---
doc_id: design.skills.window-18-deep
status: outline
last_edited_at: 2026-05-05
---

# Skill outline — window-18-deep

Window theme: 18:00 deep / practitioner briefing. Evening read for builders: notable open-source projects, research preprints, model-infra updates, and tools genuinely worth exploring or saving.

The skill drives the four stages below; both subagents are bounded helpers. Per-run intermediate artifacts live under `Runtime/runs/18/<YYYYMMDD-HHMM>/`.

## Stage 1 — Collection

Run `python -m ai_sources.core.cli config/window_18.json week` and write stdout to `Runtime/runs/18/<YYYYMMDD-HHMM>/collected.json`. The config covers developer signal (GitHub Trending), research preprints (arXiv cs.AI), agent frameworks (LangChain), developer platforms (Hugging Face), model infra (Groq, Fireworks, Cerebras, Together, Modal, Replicate, RunPod), cloud platforms (AWS ML, Microsoft AI), and technical news (MarkTechPost). `defaults.limit` is bumped to 15 because this window tolerates a deeper batch.

## Stage 2 — Screener invocation

Before invocation, run the shared helper utility `.agents/skills/_shared/scripts/strip_for_screener.py` with `collected.json` as input to produce `Runtime/runs/18/<YYYYMMDD-HHMM>/stripped.txt`. Then invoke the screener inline, passing in the prompt: the path to `collected.json`, the path to `stripped.txt`, the path where it should write `Runtime/runs/18/<YYYYMMDD-HHMM>/shortlist.json`, the `prior_recommended_dedupe_keys` from the workdir state file, and this window-specific relevance guidance:

- Window theme: deep technical and practitioner-oriented content; tools, projects, and research a builder would actually use.
- Relevance criteria: prefer notable GitHub repos with real traction or novel design; arXiv papers with concrete methods or strong empirical results; substantive infra or framework updates that change what builders can do; tutorials only if they teach something genuinely new.
- De-prioritize: trivial repo bumps, pure marketing posts, paper abstracts that promise more than they deliver, repackaged tutorials.
- Recency intuition: a one-week window is the working horizon (`since=week`); very recent items are nice but a strong project from earlier in the week is fully acceptable.

Receive the `shortlist.json` path and the screener's natural-language filtering summary.

## Stage 3 — Researcher invocation

For each record in `shortlist.json`, invoke the researcher inline with the record's content (title, url, summary, sources, published_at) embedded in the prompt plus this window-specific focus: identify the core technical claim or capability, locate concrete evidence (repo README, paper PDF, release notes, benchmark numbers), assess feasibility and maturity (working code? reproducible? license?), and surface unresolved risks (overstated benchmarks, missing eval, hidden dependencies). For `github_trending` records remember `summary` may carry HTML and `published_at` is `None`. Persist each returned briefing to `Runtime/runs/18/<YYYYMMDD-HHMM>/briefings/<record_id>.md`.

## Stage 4 — Final delivery

Synthesize a Chinese, no-markdown report into `Runtime/runs/18/<YYYYMMDD-HHMM>/output.md`: a summary paragraph ≤200 字 framing the evening's technical landscape — what builders should pay attention to and why — then a list where each item carries Title, Why-worth-reading (concrete value to a practitioner), URL. Quality over volume. Invoke the notifications CLI to default-broadcast the rendered output across all configured channels — `python -m ai_informer.notifications.cli --message-file Runtime/runs/18/<YYYYMMDD-HHMM>/output.md` — and append delivered `dedupe_key`s to prior-delivered state.
