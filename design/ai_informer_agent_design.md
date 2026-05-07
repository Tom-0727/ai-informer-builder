---
doc_id: design.ai-informer-agent.v3
status: draft
authoritative_inputs: [.claude/skills/build-a-codex-agent/SKILL.md, long-run-agent-harness/engine/scaffolds/basic-agent/{AGENTS.md,CLAUDE.md}, ai_sources/README.md, ai_sources/core/cli.py, ai_sources/core/config/sources.json, ai_sources/core/models.py, design/ai_informer_design_validation_v{1,2,3}.md]
last_edited_at: 2026-05-05
---

# AI Informer — Agent Design

## 1. Business Goal

AI Informer is a personal AI-news analyst delivering three time-aware briefings per day, prioritizing timely, useful, trustworthy information over volume. 08:00 = urgency (major model/product releases from labs like OpenAI, Anthropic). 12:00 = conceptual (essays, opinions, "what's next for agents"). 18:00 = practitioner (GitHub Trending, papers, tools worth exploring). Output volume is up to 15 items per window (no strict floor; quality first).

Output text structure is two parts. (a) A summary paragraph of ≤200 字 framing the window's overall picture. (b) A news list where each item carries exactly three fields: Title, Why-worth-reading, URL. Output is Chinese, no markdown.

## 2. Architecture Design

AI Informer is a vertical Codex agent built on the `basic-agent` scaffold (`@openai/codex-sdk`, one-shot runs). The three layers from the `build-a-codex-agent` skill:

- **Main Codex agent** — durable owner. Lives in `AGENTS.md`. Owns identity, the user relationship, final pick, and Chinese delivery rendering.
- **Skills** — reusable workflow recipes under `.agents/skills/<name>/SKILL.md`. Auto-discovered. The three per-window skills orchestrate the four-stage flow; they hold no business judgment.
- **Subagents** — under `.codex/agents/<name>.toml`. The main agent invokes them inline within a single Codex run as bounded helpers; they return natural-language results and never commit a recommendation.

Two subagents: `screener` (file-mediated coarse semantic filter over the candidate batch) and `researcher` (per-shortlist evidence enrichment). Both are inline Codex subagent calls — not skills, not peer agents, not separate runtimes.

## 3. Runtime Mechanism

§3 describes the abstract business-logic mechanism, not a code-level runtime. The implementation scaffold is `basic-agent`; customization happens in three places — `AGENTS.md` (overall business logic), `.agents/skills/` (per-window concrete execution logic plus a shared utility under `.agents/skills/_shared/scripts/`), and `.codex/agents/*.toml` (the screener and researcher subagents).

OS cron (no scheduler daemon, per `basic-agent`) fires three crontab entries at 08/12/18 local. Each fire injects a wake-up message into the main Codex agent of the form `现在是 HH:00，请执行 <window 主题> 推荐` (e.g., `现在是 08:00，请执行今晨高优先级资讯推荐`). The main agent parses the timestamp, picks the matching window skill, and runs one independent Codex run with no in-process state carried between runs. Cross-run state (prior-delivered `dedupe_key` set) lives as plain files in the workdir.

Per run, the main agent creates an isolated intermediate-artifacts directory `Runtime/runs/<window>/<YYYYMMDD-HHMM>/` and drives the four file-mediated stages from the active window skill:

1. **Collect** — invoke `python -m ai_sources.core.cli config/window_<NN>.json <today|week|month>` per the skill (`since` is `today` for 08 and 12, `week` for 18) and write stdout to `<run-dir>/collected.json`.
2. **Strip + screen** — run the shared helper utility `.agents/skills/_shared/scripts/strip_for_screener.py` to read `collected.json` and emit `<run-dir>/stripped.txt`, one line per record carrying semantic-signal fields (title, joined sources, joined source_categories, published_at, summary excerpt, record_id; `raw_items` excluded). Invoke the `screener` subagent with paths to `collected.json`, `stripped.txt`, the target `shortlist.json`, the window-specific relevance guidance, and `prior_recommended_dedupe_keys`. The screener writes `<run-dir>/shortlist.json` and returns a natural-language filtering summary.
3. **Research** — for each shortlisted record, invoke the `researcher` subagent inline with the record's content (title, url, summary, sources, published_at) embedded in the prompt plus the window-specific focus pointers. Persist each returned briefing to `<run-dir>/briefings/<record_id>.md`.
4. **Deliver** — synthesize a Chinese, no-markdown report into `<run-dir>/output.md` (≤200 字 summary paragraph + per-item Title / Why-worth-reading / URL list, up to 15 items, quality over volume), invoke the notifications CLI on `output.md` (default broadcast across configured channels — see §6 and `design/notifications.md`), and append delivered `dedupe_key`s to prior-delivered state.

## 4. Subagent Contracts

The `screener` is invoked file-mediated. The main agent's prompt names the path to the run's `collected.json`, the path to a pre-stripped `stripped.txt` produced by the shared helper utility, the target path for `shortlist.json`, the window's relevance guidance in plain prose, and `prior_recommended_dedupe_keys`. The screener reads `stripped.txt`, performs LLM-driven semantic filtering against the guidance, drops anything already delivered, writes `shortlist.json` (each kept entry carries `record_id` plus a short natural-language reason), and returns the file path together with a natural-language summary of what was filtered. File-read and file-write capability are granted; no web fetch, no message I/O.

The `researcher` is invoked prompt-mediated. The main agent's prompt embeds one record's content (title, url, summary, sources, published_at) plus window-specific focus pointers; no file path is passed in. The researcher performs a small number of bounded same-host or configured-source GET fetches (only as needed) to confirm what is genuinely new, distinguish substantive from hyped framing, and capture concrete evidence URLs. It returns a concise but substantive natural-language briefing covering what the news actually is, why it matters / impact, evidence pointers actually consulted, and residual hype or credibility concerns. Web-read allowed within that bound; no file writes, no message I/O.

Detailed prose contracts for each subagent live in `design/subagents/{screener,researcher}.md`. Window-specific theme, relevance criteria, recency intuition, and researcher focus live in `design/skills/window-*.md` and are passed to the subagents at invocation time.

## 5. Per-Window Skills

`.agents/skills/window-08-urgency/`, `window-12-opinion/`, `window-18-deep/` each ship a `SKILL.md` carrying the window's semantic guidance plus a pointer to the per-window source config under `config/`. The skill orchestrates the four file-mediated stages above; subagents only see the prompt-borne semantic guidance the skill supplies.

A cross-cutting `recommendation-output-format` skill encodes the OUTPUT TEXT STRUCTURE: the two-part shape — (a) Chinese summary paragraph ≤200 words, (b) news list where each item is exactly `Title` / `Why-worth-reading` / `URL`. Pure function over final picks → output text; no I/O, no business judgment.

Per-window source configs live in `config/window_*.json`; per-window skill outlines and subagent instruction outlines live in `design/skills/window-*.md` and `design/subagents/{screener,researcher}.md`.

## 6. Data Invariants (validation-grounded defaults)

- `CanonicalCandidateRecord` fields are fixed: `record_id, dedupe_key, title, url, published_at, summary, tags, source_item_ids, sources, source_categories, merged_item_count, raw_items`. Engagement signals live at `raw_items[*].source_metrics`; the design does not invent fields (v1).
- `published_at` is RFC 822 when present. `github_trending` returns `None` batch-wide; `hacker_news` canonical records also return `None` (epoch at `raw_items[0].raw.time`). Recency for these adapters falls back to `collected_at` or the adapter-specific raw time (v1, v3).
- Cross-source merging is empirically rare (`merged_item_count > 1` fired 0/69 across two `quick/limit=3` multi-source runs, v2). HN engagement metrics in `raw_items[*].source_metrics` are available to the screener as semantic context for the 08 window. `reddit-ai` adapter health is tracked separately (HTTP 403 in v3) and is not a design clause.
- `github_trending` `summary` may carry HTML; consumers must strip before length-scoring (v1). HN `summary` is already plaintext (v3).
- Per-run intermediate artifacts live under `Runtime/runs/<window>/<YYYYMMDD-HHMM>/` and contain `collected.json`, `stripped.txt`, `shortlist.json`, `briefings/<record_id>.md`, and `output.md`; runs are isolated and never reuse a directory.
- Delivery channel: configured channels are authored in exactly one place — `config/notifications.json` (single source of truth). At end of each run the main agent invokes the notifications CLI which broadcasts to every configured channel by default; a `--channel <name>` flag restricts to one. Module/CLI/adapter/extensibility details live in `design/notifications.md`.
- Side-effect ownership: only the main agent writes to prior-delivered state and invokes the notifications CLI. The screener owns its `shortlist.json` write under the run dir; the researcher is read-only over its inputs and web-reads within the bounded fetch constraint.
