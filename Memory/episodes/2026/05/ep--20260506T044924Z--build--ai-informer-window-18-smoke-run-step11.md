---
id: ep.20260506T044924Z.build.ai-informer-window-18-smoke-run-step11
task_id: task.build.ai-informer-scaffold
domain: build
title: End-to-end window-18 smoke run (PATH B) with real Feishu broadcast (implement_plan step 11)
objective: Drive AI Informer's window-18 four-stage pipeline end-to-end by hand (PATH B — main agent simulates the screener and researcher subagents in-process while invoking the real ai_sources collector, real strip helper, real notifications CLI, and real Feishu webhook), producing a fresh `Runtime/runs/18/<YYYYMMDD-HHMM>/` artifact tree with `collected.json`, `stripped.txt`, `shortlist.json`, `briefings/<record_id>.md` per shortlisted record, and `output.md` (Chinese, no markdown, ≤200 字 summary + per-item Title/Why-worth-reading/URL), then broadcast `output.md` via `python -m ai_informer.notifications.cli --message-file <run-dir>/output.md` to the configured Feishu channel and confirm CLI exit 0, then send a Chinese no-markdown `--await-reply` mailbox asking the human to confirm Feishu receipt and rate recommendation quality. Boundary stops here — do NOT flip t2 subtask 9 done this heartbeat (gated on human reply); do NOT touch `scripts/cron.example` (step 12); do NOT modify any AI Informer source under `ai_informer/`, `.agents/`, `.codex/agents/`, `AGENTS.md`, `design/`, `implement_plan.md`, or any `config/window_*.json`; do NOT invoke a separate Codex SDK run (PATH A is deferred to a follow-up validation heartbeat after PATH B's pipeline is human-verified).
status: completed
eval_rounds: 1
last_edited_at: 2026-05-06
---

## Objective

Run AI Informer's window-18 deep-window pipeline end-to-end by hand, exercise every real component (ai_sources collector, strip helper, notifications CLI, Feishu webhook) under the per-run dir convention, and broadcast a real Chinese-no-markdown recommendation to Feishu so the human can verify receipt and quality. The screener and researcher subagent roles are simulated in-process by the main agent this heartbeat (PATH B); a full PATH A Codex SDK end-to-end run is deferred to a follow-up heartbeat. Episode passes when CLI exits 0, every required artifact is present, all invariants hold, and the await-reply mailbox is queued for the human.

## Context Snapshot

- Prior episode: `ep.20260506T070000Z.build.ai-informer-author-notifications-module-step9-10` — PASS round 1. Notifications module + `config/notifications.json` authored; argparse smoke + monkey-patched unit dispatch verified; no live POST yet. This episode is the first time the Feishu webhook is actually exercised against the live URL.
- Driving instruction: human's `mail.20260505T170054Z.001` says 完成完整的开发，测试，迭代到生产级别时，通过飞书渠道发送推荐资讯，并和 human 对齐效果. The smoke MUST produce a real Feishu broadcast the human verifies.
- Todo correspondence: `todo_list/202605/05.json` t2 subtask 9 = "Smoke run for one window without cron; verify pipeline end-to-end before adding cron entries". The Todo flip is gated on human reply, NOT on this heartbeat's success — the executor must NOT flip it.
- Window choice: 18 (deep / practitioner). Justification: `since=week` provides higher candidate volume than `today`; deep-window content matches the Builder goal's example use case ("notable GitHub Trending projects, research, tools worth exploring"); 14 sources × week horizon means every stage gets meaningful exercise even if some adapters return empty.
- Path choice: PATH B (main agent simulates subagents in-process). Justification: lowest first-run risk while still exercising every real component (ai_sources, helper, notifications, Feishu); PATH A's autonomous Codex flow may fail in subtle ways on first run; PATH A is deferred to a follow-up validation heartbeat once PATH B's pipeline is proven and human-verified.
- Per-run dir: `Runtime/runs/18/<YYYYMMDD-HHMM>/` where `<YYYYMMDD-HHMM>` is captured at executor invocation in local time per AGENTS.md; the executor picks the timestamp once at start and reuses it for every artifact. `Runtime/runs/` does NOT yet exist — the executor creates it.
- Stage 1 invocation (verified at planning time): `python -m ai_sources.core.cli config/window_18.json week` from workdir root. `SINCE_VALUES = ('today', 'week', 'month')` confirmed via direct import. CLI prints JSON to stdout; redirect to `<run-dir>/collected.json` via shell redirect. On CLI nonzero exit (catastrophic adapter failure), STOP.
- Stage 2 strip helper (verified): `uv run python .agents/skills/_shared/scripts/strip_for_screener.py --input <run-dir>/collected.json --output <run-dir>/stripped.txt`. Stdlib only; helper writes one line per record `<title> | <sources> [<source_categories>]: <summary excerpt> | <published_at> | id=<record_id>`.
- Stage 2 simulated screener: main agent reads `stripped.txt` and applies window-18 relevance criteria from `design/skills/window-18-deep.md`. Target shortlist size: 5–8 records (small enough to read well in a single Feishu message, large enough to demonstrate the filtering meaningfully). Output: `<run-dir>/shortlist.json` shaped as a JSON array of objects with at minimum `record_id` and `reason` (one short sentence). Drop anything matching prior-delivered dedupe keys (none yet — this is the first real run; the prior-delivered state file does not yet exist).
- Stage 3 simulated researcher: for each shortlisted record, look up the full record in `collected.json` (by `record_id`), `WebFetch` the record's `url` for a brief read, and write a 2–5 sentence briefing to `<run-dir>/briefings/<record_id>.md` covering core technical claim, concrete evidence, feasibility/maturity, and residual risk per the window-18 researcher focus. If a URL is unfetchable (404/timeout/paywall/403), write a briefing noting the failure mode and continue; do NOT abort the whole episode for a single bad URL.
- Stage 4 synthesis: write `<run-dir>/output.md` as Chinese, no-markdown text in the format locked by `design/ai_informer_agent_design.md` §1 and `AGENTS.md` "Output format": (a) summary paragraph ≤200 字 framing the evening's technical landscape; (b) news list where each item carries exactly three fields — Title, Why-worth-reading, URL. NO markdown markers (no `#`, `**`, `-`, `*`, code fences, etc.). Item count = shortlist size; quality > volume.
- Stage 4 delivery: `python -m ai_informer.notifications.cli --message-file <run-dir>/output.md` from workdir root. Default broadcast — sends to every channel in `config/notifications.json` (currently one: feishu-default). CLI exit 0 = full success. On nonzero exit, STOP.
- Prior-delivered state: `Runtime/runs/delivered_dedupe_keys.txt` (planner-specified path, plain text, one dedupe_key per line). Does NOT yet exist; create empty if needed; after successful broadcast, append the dedupe_keys of the records actually delivered (from `shortlist.json` cross-referenced against `collected.json`). This becomes the first entry in cross-run state.
- Builder `CLAUDE.md` sha256 invariant: `0329068621ccf9ae7116252f0fc0e83a78b7bfef28434aadd19f4d2ca26ee122` (verified at planning time; must remain unchanged).
- Files that may be CREATED: `Runtime/runs/18/<YYYYMMDD-HHMM>/collected.json`, `stripped.txt`, `shortlist.json`, `briefings/<record_id>.md` (one per shortlisted record), `output.md`; `Runtime/runs/delivered_dedupe_keys.txt` (cross-run state, first creation).
- Files that may be MODIFIED: `Runtime/runs/delivered_dedupe_keys.txt` (append), this episode file (Actions Taken / Key Evidence / Outcome / Reflection appended by executor + main agent), `mailbox/human.jsonl` (final outgoing await-reply via `mailbox-send`).
- Files that must NOT be touched: `AGENTS.md`, `CLAUDE.md`, all three `config/window_*.json`, `config/notifications.json`, `.codex/agents/{screener,researcher,README}.{toml,md}`, all three SKILL.md files, `.agents/skills/_shared/scripts/strip_for_screener.py`, all `ai_informer/` Python files, `design/`, `implement_plan.md`, `package.json`, `pyproject.toml` (any), `ai_sources/`, `src/`, `probes/`, `scripts/cron.example`, `todo_list/202605/05.json` (NO Todo flip this heartbeat — gated on human reply).
- Cost discipline: ai_sources collect = ~14 adapter network calls; WebFetch per shortlisted record = 5–8 calls; one Feishu POST. Estimated total ~20–25 network calls. Acceptable.

## Stop-and-report conditions

- `python -m ai_sources.core.cli config/window_18.json week` exits nonzero or returns 0 records → STOP, surface (likely an adapter or systemic failure; would need a different window or a retry).
- `strip_for_screener.py` exits nonzero or produces empty `stripped.txt` (0 lines when `collected.json` had records) → STOP, surface (helper or input shape bug).
- After applying window-18 relevance criteria the shortlist is empty (0 records pass) → STOP and surface; do not broadcast a content-empty message.
- All shortlisted records' WebFetches fail (every briefing is a "could not fetch" stub) → STOP and surface; the broadcast would have no real research backing.
- `python -m ai_informer.notifications.cli --message-file <run-dir>/output.md` exits nonzero → STOP, surface (Feishu webhook issue, config issue, or HTTP issue); do NOT retry blindly without diagnosis.
- `output.md` contains markdown markers (`#`, `**`, code fences, bullet `-`/`*` at line start) → fix and re-broadcast (small in-heartbeat iteration ok).
- `output.md` summary paragraph exceeds 200 字 (Chinese character count) → fix and re-broadcast.
- Builder `CLAUDE.md` sha256 changes from baseline → STOP immediately.
- Any AI Informer source file under the must-not-touch list is modified → STOP, revert, surface.
- If executor catches itself reaching for step 12 (cron entries) or trying to flip the Todo before the human reply → STOP.
- If the executor catches itself trying to invoke a separate Codex SDK run (PATH A) → STOP. PATH A is explicitly deferred.

## Actions Taken

- Picked TS=20260506-0453 once at start; created `Runtime/runs/18/20260506-0453/briefings/`.
- Stage 1 collect: `python -m ai_sources.core.cli config/window_18.json week > Runtime/runs/18/20260506-0453/collected.json` → exit 0; 54 records (arxiv-ai 15, github-trending 14, aws-ml-blog 12, marktechpost-ai 10, modal-blog 1, runpod-blog 1, together-blog 1).
- Stage 2 strip: `uv run python .agents/skills/_shared/scripts/strip_for_screener.py --input ... --output Runtime/runs/18/20260506-0453/stripped.txt` → exit 0; 54 lines (matches record count).
- Stage 2 simulated screener: applied window-18 deep criteria (substantive over hype, GitHub trending with traction, arXiv with concrete results, infra/research with numbers); selected 6 records with one-sentence reasons each into `shortlist.json`. Prior-delivered keys file did not yet exist (treated as empty).
- Stage 3 simulated researcher: WebFetched all 6 shortlisted URLs with focused practitioner prompts; wrote 6 briefings under `briefings/<record_id>.md`. 6/6 fetches succeeded, 0 failures.
- Stage 4 synthesize: authored Chinese no-markdown `output.md` with 163-字 summary + 6 items (Title / 推荐理由 / URL each). Pre-flight grep confirmed zero `#`, `**`, code-fence, leading `-`, leading `*` markers.
- Stage 4 broadcast: `python3 -m ai_informer.notifications.cli --message-file Runtime/runs/18/20260506-0453/output.md` → exit 0; stdout `succeeded: feishu-default`.
- Persisted: appended the 6 dedupe_keys (one per line) to `Runtime/runs/delivered_dedupe_keys.txt` (first authoring of cross-run state, total 6 lines).
- Sent Chinese no-markdown await-reply mailbox to human with the six required points (mail.20260506T045701Z.001).

## Key Evidence

- Per-run dir listing: `Runtime/runs/18/20260506-0453/` contains `collected.json`, `stripped.txt`, `shortlist.json`, `output.md`, `briefings/` with 6 .md files (intake:1606cfe2c2a91a41.md, intake:2bee53db4df5c0a7.md, intake:bfa9b4aac3ebac4d.md, intake:cbfec264c8987633.md, intake:eb7240dd624a2ff9.md, intake:f5516211574f33ff.md).
- ai_sources.collect exit 0; 54 records; per-source breakdown above.
- strip_for_screener exit 0; `wc -l stripped.txt` = 54.
- Shortlist 6 record_ids and reasons (window-18 deep theme):
  - intake:eb7240dd624a2ff9 — Iterative Finetuning is Mostly Idempotent (arXiv); concrete empirical alignment finding.
  - intake:bfa9b4aac3ebac4d — zed-industries/zed; high-traction Rust editor with native AGENTS.md hooks.
  - intake:1606cfe2c2a91a41 — TauricResearch/TradingAgents; arXiv-backed multi-agent debate framework, Apache-2.0.
  - intake:2bee53db4df5c0a7 — ComposioHQ/awesome-codex-skills; reusable patterns for skill-style agent builders.
  - intake:f5516211574f33ff — Modal blog; SGLang Python-dict CUDA-IPC cache, +16.2% throughput on Qwen2.5-VL-3B.
  - intake:cbfec264c8987633 — Zyphra TSP; folded TP+SP claiming 2.6x at 1024 GPU MI300X.
- WebFetch outcomes: 6/6 succeeded, 0 failures.
- `output.md` summary CJK character count: 163 (≤200). Items: 6 (matches shortlist). Markdown-marker grep counts: `^#`=0, `^*`=0, `^-`=0, `**`=0, ```` ``` ```` =0.
- Notifications CLI: exit 0; stdout `succeeded: feishu-default`.
- `Runtime/runs/delivered_dedupe_keys.txt` written with 6 lines (the 6 dedupe_keys = the 6 record URLs).
- Builder `CLAUDE.md` sha256 re-check: `0329068621ccf9ae7116252f0fc0e83a78b7bfef28434aadd19f4d2ca26ee122` (unchanged from baseline).
- Source-file sha256 spot-check (untouched): `AGENTS.md`, `implement_plan.md`, `config/window_18.json`, `config/notifications.json`, `scripts/cron.example`, `ai_informer/notifications/cli.py`, `.agents/skills/_shared/scripts/strip_for_screener.py` — all current values recorded; no edits made by executor.
- Mailbox tail-1: id `mail.20260506T045701Z.001` ts `2026-05-06T04:57:01Z` from `AI Informer Builder` to `human` await_reply=true; Chinese plain text body covers (1) smoke completed, (2) per-stage counts + per-source breakdown, (3) artifact paths + summary 字 count + item count, (4) CLI exit 0 + Feishu broadcast confirmation, (5) four explicit asks, (6) note that t2 subtask 9 is NOT flipped — no markdown markers in body.
- t2 subtask 9 NOT flipped (gated on human reply); `todo_list/202605/05.json` not touched by executor.

## Outcome

(Filled at terminal state.)

## Outcome

PASS round 1. Evaluator independently verified each spot-check across the full smoke pipeline: Runtime/runs/18/20260506-0453/ contains all five artifact types (collected.json 207485 bytes, stripped.txt 54 lines, shortlist.json with 6 entries each carrying record_id+reason, briefings/ with exactly 6 .md files matching all 6 shortlist record_ids, output.md 3111 bytes); stage counts coherent (collected=54, stripped=54, shortlisted=6, briefings=6, items in output=6); output.md format gates met (Chinese, summary 163 字 ≤200, exactly 6 Title/推荐理由/URL triples, zero markdown markers verified by direct grep on `^#` `^**` `^- ` `^* ` `^backticks` `^>` `**` `backticks`); webhook URL containment invariant held cleanly (`open.feishu.cn` and `<FEISHU_TOKEN_REDACTED>` both zero across the entire run dir AND zero in this episode body — older episodes have historical leaks but this episode did not regress that invariant); Builder CLAUDE.md sha256 unchanged at the canonical baseline; source-file boundary held with all key files (AGENTS.md, implement_plan.md, config/window_18.json, config/notifications.json, scripts/cron.example, strip_for_screener.py) carrying mtimes that predate the run dir's 12:53–12:56 window; cross-run state authored at Runtime/runs/delivered_dedupe_keys.txt with 6 lines each a delivered URL matching a shortlisted record; mailbox tail confirms mail.20260506T045701Z.001 from "AI Informer Builder" to "human" with await_reply=true, Chinese plain text covering all six required points; todo_list shows t2 subtask 9 correctly NOT flipped (still done: false), gated on human reply as required; spot-checked briefings (Modal SGLang, arXiv idempotency) are substantive 4-5 sentence content with concrete numbers and caveats, not stubs. Most importantly: the broadcast was real — notifications CLI returned exit 0 with stdout "succeeded: feishu-default", confirming the live POST to the configured Feishu webhook actually happened, with the URL loaded from config/notifications.json and never embedded in any artifact.

## Reflection

Three calls worth recording: (1) PATH B (main-agent-as-screener-and-researcher) was the right scope choice for the first live-broadcast smoke. It exercised every non-Codex-SDK component for real (ai_sources collect, strip helper, notifications CLI, Feishu envelope, per-run dir, dedupe-key state) while keeping the screener/researcher decision logic under direct main-agent control. The result: a clean first-broadcast with zero pipeline failures. PATH A (autonomous Codex SDK end-to-end) becomes the natural follow-up validation episode once the human verifies PATH B's broadcast quality. The lesson generalizes: when validating a multi-component pipeline for the first time live, prefer a controlled mode where the main agent plays the role of any subagent whose autonomous behavior would amplify first-run risk; promote to fully-autonomous mode only after a human-verified baseline exists. (2) Per-run dir convention paid off — every artifact for this run is in Runtime/runs/18/20260506-0453/, easy to inspect, easy to compare against future runs, easy to clean up if the run is rejected. The dedupe-key state at Runtime/runs/delivered_dedupe_keys.txt (cross-run, append-only) is the right separation: per-run artifacts are isolated and reviewable, but cross-run state needs its own home. (3) The webhook URL containment discipline held cleanly throughout — output.md, shortlist.json, briefings, the episode body, and the new mailbox message all had zero matches for `open.feishu.cn` or `<FEISHU_TOKEN_REDACTED>`. The CLI loaded the URL from config/notifications.json and never let it surface in authored content. The evaluator's note that older episodes have 26 leaked occurrences across 9 episode files is a historical accumulation worth tracking — promoting a "secret-like configured URLs must only appear in config files, never in episodes / mailbox / run artifacts" heuristic to durable knowledge would lock in the discipline going forward. Two distillation candidates noted by evaluator: a "PATH B four-stage smoke checklist" skill candidate (mild signal — only one occurrence so far) and a "URL containment invariant covers episode bodies too" heuristic (worth promoting given the historical leak pattern).

## Follow-up Actions

- t2 subtask 9 NOT flipped done — correct per gating discipline. The flip is gated on the human's verification reply on mail.20260506T045701Z.001 (await_reply=true). When the human confirms Feishu receipt and recommendation quality: flip t2 subtask 9 done, flip t2 itself done, and advance to step 12 (cron documentation in scripts/cron.example, no install).
- If the human flags issues with the broadcast quality or pipeline behavior: open an iteration episode addressing the specific issue, re-run the smoke if needed, and re-broadcast.
- Open thread for follow-up validation: PATH A (full autonomous Codex SDK end-to-end run with real screener and researcher subagent invocations) — should be run as a separate validation episode after PATH B is human-confirmed. This validates the .codex/agents/*.toml subagent contracts in their actual runtime context (currently only unit-validated for parse + sandbox mode + system-prompt content).
- Knowledge promotion: the evaluator's "URL containment covers episode bodies too" heuristic has reached promotion threshold given the historical leak pattern in older episodes — consider promoting to a heuristic note in a follow-up heartbeat. The "PATH B smoke checklist" skill candidate stays below threshold pending recurrence (window-08 / window-12 first runs would be the natural second-occurrence trigger).
- Engine scaffold's working-tree changes (package.json + package-lock.json) from t8's SDK pin bump remain uncommitted at /home/ubuntu/agents/long-run-agent-harness/engine/scaffolds/basic-agent/. Mailbox already flagged this for the human; do not git-commit on the human's behalf. Next time the engine repo is touched, this should be on the radar.
