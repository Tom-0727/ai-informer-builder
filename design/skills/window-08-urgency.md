---
doc_id: design.skills.window-08-urgency
status: outline
last_edited_at: 2026-05-05
---

# Skill outline — window-08-urgency

Window theme: 08:00 urgency briefing. The user is starting the day and needs to immediately know about high-impact, time-sensitive AI events from the last ~12–24 hours: official model or product releases, major lab announcements, and headline industry moves.

The skill drives the four stages below; both subagents are bounded helpers. Per-run intermediate artifacts live under `Runtime/runs/08/<YYYYMMDD-HHMM>/`.

## Stage 1 — Collection

Run `python -m ai_sources.core.cli config/window_08.json today` and write stdout to `Runtime/runs/08/<YYYYMMDD-HHMM>/collected.json`. The config covers official labs (OpenAI, Anthropic, Google AI, DeepMind, Meta AI, Mistral), AI hardware (NVIDIA, AI-keyword filtered), community/product signal (Hacker News, Product Hunt), and industry news (TechCrunch AI, VentureBeat AI, AI News).

## Stage 2 — Screener invocation

Before invocation, run the shared helper utility `.agents/skills/_shared/scripts/strip_for_screener.py` with `collected.json` as input to produce `Runtime/runs/08/<YYYYMMDD-HHMM>/stripped.txt`. Then invoke the screener inline, passing in the prompt: the path to `collected.json`, the path to `stripped.txt`, the path where it should write `Runtime/runs/08/<YYYYMMDD-HHMM>/shortlist.json`, the `prior_recommended_dedupe_keys` from the workdir state file, and this window-specific relevance guidance:

- Window theme: high-urgency, time-sensitive AI updates the user should know immediately.
- Relevance criteria: prefer official model or product releases from major labs; major capability announcements; significant industry moves with real-world impact today; product launches with a working artifact, not vapor.
- De-prioritize: opinion essays, generic listicles, low-impact funding rumors, marketing reposts.
- Recency intuition: items from roughly the last 12–24 hours are ideal; older items only if the news first surfaced today.
- Hacker News engagement metrics in `raw_items[*].source_metrics` are available as semantic context — high engagement on an AI-relevant story is a positive signal, not a hard rule.

Receive the `shortlist.json` path and the screener's natural-language filtering summary.

## Stage 3 — Researcher invocation

For each record in `shortlist.json`, invoke the researcher inline with the record's content (title, url, summary, sources, published_at) embedded in the prompt plus this window-specific focus: confirm whether this is a primary-source release vs reblog, identify what is genuinely new (model, capability, pricing, availability), capture concrete evidence URLs, and surface unresolved risks (claim vs benchmark, regional rollout caveats, paywall). Persist each returned briefing to `Runtime/runs/08/<YYYYMMDD-HHMM>/briefings/<record_id>.md`.

## Stage 4 — Final delivery

Synthesize a Chinese, no-markdown report into `Runtime/runs/08/<YYYYMMDD-HHMM>/output.md`: a summary paragraph ≤200 字 framing the morning's overall picture, then a list where each item carries Title, Why-worth-reading, URL. Quality over volume. Invoke the notifications CLI to default-broadcast the rendered output across all configured channels — `python -m ai_informer.notifications.cli --message-file Runtime/runs/08/<YYYYMMDD-HHMM>/output.md` — and append delivered `dedupe_key`s to prior-delivered state.
