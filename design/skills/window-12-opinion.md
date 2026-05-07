---
doc_id: design.skills.window-12-opinion
status: outline
last_edited_at: 2026-05-05
---

# Skill outline — window-12-opinion

Window theme: 12:00 opinion and commentary briefing. Midday read for forward-looking essays, expert analysis, and conceptual pieces — "what's next for agents", framing of recent shifts, evergreen ideas worth thinking about.

The skill drives the four stages below; both subagents are bounded helpers. Per-run intermediate artifacts live under `Runtime/runs/12/<YYYYMMDD-HHMM>/`.

## Stage 1 — Collection

Run `python -m ai_sources.core.cli config/window_12.json today` and write stdout to `Runtime/runs/12/<YYYYMMDD-HHMM>/collected.json`. The config covers expert analysis (Sebastian Raschka), research commentary (The Gradient), editorial practice (Towards AI, Towards Data Science), Chinese business commentary (36Kr), global AI news with editorial framing (Synced), and curated newsletters (The Rundown AI).

## Stage 2 — Screener invocation

Before invocation, run the shared helper utility `.agents/skills/_shared/scripts/strip_for_screener.py` with `collected.json` as input to produce `Runtime/runs/12/<YYYYMMDD-HHMM>/stripped.txt`. Then invoke the screener inline, passing in the prompt: the path to `collected.json`, the path to `stripped.txt`, the path where it should write `Runtime/runs/12/<YYYYMMDD-HHMM>/shortlist.json`, the `prior_recommended_dedupe_keys` from the workdir state file, and this window-specific relevance guidance:

- Window theme: opinionated and conceptual content — essays, blogs, forward-looking discussions about where AI is heading.
- Relevance criteria: prefer pieces with a clear thesis, original framing, or expert perspective; conceptual depth over news recap; "what does this mean / what comes next" angle; substantive Chinese analysis (36Kr) when present.
- De-prioritize: pure news rewrites, vendor marketing dressed as opinion, link-list roundups without a thesis.
- Recency intuition: today's pieces are ideal but a strong essay from the last 1–3 days is acceptable if it has not been delivered yet.

Receive the `shortlist.json` path and the screener's natural-language filtering summary.

## Stage 3 — Researcher invocation

For each record in `shortlist.json`, invoke the researcher inline with the record's content (title, url, summary, sources, published_at) embedded in the prompt plus this window-specific focus: extract the central thesis and the supporting argument in two or three sentences, identify the author's stance and any obvious counter-positions worth flagging, capture the canonical post URL, and note unresolved risks (speculative claims, conflict of interest, evidence quality). Persist each returned briefing to `Runtime/runs/12/<YYYYMMDD-HHMM>/briefings/<record_id>.md`.

## Stage 4 — Final delivery

Synthesize a Chinese, no-markdown report into `Runtime/runs/12/<YYYYMMDD-HHMM>/output.md`: a summary paragraph ≤200 字 framing the midday discourse — what ideas are circulating, what tensions or debates the picks reflect — then a list where each item carries Title, Why-worth-reading (capturing the thesis, not just the topic), URL. Quality over volume. Invoke the notifications CLI to default-broadcast the rendered output across all configured channels — `python -m ai_informer.notifications.cli --message-file Runtime/runs/12/<YYYYMMDD-HHMM>/output.md` — and append delivered `dedupe_key`s to prior-delivered state.
