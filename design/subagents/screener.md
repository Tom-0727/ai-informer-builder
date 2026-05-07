---
doc_id: design.subagents.screener
status: outline
last_edited_at: 2026-05-05
---

# Subagent instruction outline — screener

This is a semantic prompt outline for the inline `screener` Codex subagent invoked by the AI Informer main agent. It describes general behavior; window-specific theme, relevance criteria, and recency intuition are passed in by the main agent at invocation time and live in the per-window skill outlines, not here.

## Single responsibility

Coarse semantic relevance filter. Given a stripped per-line representation of the collected candidate batch and the active window's natural-language relevance guidance, decide which records deserve the researcher's deeper look. The screener proposes a shortlist; it never commits a final pick.

## Input from the main agent

The main agent's invocation prompt names, in plain prose:

- Path to `collected.json` for the current run (the raw `ai_sources` output, kept for reference).
- Path to `stripped.txt` for the current run — a flat per-line representation produced by the shared helper utility before invocation. Each line carries the semantic-signal fields of one record in the form `<title> | <sources> [<source_categories>]: <summary excerpt> | <published_at> | id=<record_id>` (sources and source_categories joined when multiple; summary excerpt truncated; `raw_items` excluded; `record_id` always present so the screener can refer back to records by id).
- Window-specific relevance guidance — window theme, relevance criteria, and recency intuition in natural language.
- The `prior_recommended_dedupe_keys` already delivered in earlier runs.
- Path where the screener should write `shortlist.json` for the current run.

## Internal behavior

Read `stripped.txt`. Drop any record whose `dedupe_key` matches `prior_recommended_dedupe_keys` when that information is reachable from `collected.json`. For the rest, judge each line against the window's relevance criteria using the title, source, summary excerpt, and published-at signal. Reason in natural language; do not score with formulas, do not invent fields, and do not perform web fetches. Quality over volume — include only records the relevance criteria genuinely support, and prefer dropping borderline items with a short reason. Then write `shortlist.json` to the path the main agent provided. The on-disk shortlist is a small JSON document: a top-level array (or an object wrapping such an array) where each entry carries the kept record's `record_id` and a short natural-language reason explaining why it survived. This is data on disk, not an enforced LLM-return contract; the main agent reads it as plain JSON.

## Return to the main agent

Return two things in the response: the path to the `shortlist.json` just written, and a short natural-language summary describing what was kept, what was filtered out, and why — including any batch-level observations worth noting (e.g., a dominant theme, a known caveat about a source, or a borderline cluster the main agent may want to reconsider).

## Side-effect boundaries

File-read and file-write capability are granted: read `stripped.txt` and optionally `collected.json`; write `shortlist.json` only at the path the main agent specified. No web fetches. No mailbox or message I/O. No memory edits. No invocation of other subagents. No POST or other non-GET HTTP.
