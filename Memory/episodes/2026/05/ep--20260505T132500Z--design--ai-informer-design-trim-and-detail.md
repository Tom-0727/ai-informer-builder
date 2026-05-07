---
id: ep.20260505T132500Z.design.ai-informer-design-trim-and-detail
task_id: task.design.ai-informer-agent-design
domain: design
title: AI Informer design — trim numerical-knob language and HN amplifier, then produce per-window configs and 5 outline files
objective: Apply T1 (delete tunable rubric parameters and reshape window_profile to pure semantic guidance) and T2 (delete HN reverse-lookup amplifier; HN becomes a normal 08-window source) to design/ai_informer_agent_design.md, then produce 8 new artifacts (3 per-window source configs under config/, 3 per-window skill outlines under design/skills/, 2 subagent instruction outlines under design/subagents/) grounded in the updated ai_sources interface, and send a Chinese no-markdown mailbox reply via mailbox-send --await-reply summarizing the trim, listing pointers to the 8 new files, and requesting approval before scaffolding.
status: completed
eval_rounds: 1
last_edited_at: 2026-05-05
---

## Objective

Apply the two trim instructions to the main design doc, generate the eight detail-design artifacts (3 per-window source configs + 3 per-window skill outlines + 2 subagent instruction outlines), and send a Chinese no-markdown mailbox reply with --await-reply that points the human to all eight artifacts and asks for approval before scaffolding.

## Context Snapshot

- Driving input: mail.20260505T132212Z.001 (human, 2026-05-05). Two trims (T1, T2) on the existing design + three detail-design tasks (D1 per-window source-set configs, D2 per-window skill 4-stage outlines, D3 screener/researcher instruction outlines).
- Prior episode: ep.20260505T110000Z.design.ai-informer-design-revision-v2 (PASS r1) produced the current 73-line design that still carries the numerical knobs in §3 step 1, §3 step 4, §3 step 5, §4.1 task packet, §4.2 task packet, §5 per-window skill description, plus the HN reverse-lookup amplifier in §6.
- ai_sources is updated. Executor MUST read /home/ubuntu/agents/builders/ai-informer-builder/ai_sources/README.md FIRST. The new interface: CLI takes only `(sources_config_json, today|week|month)`; the canonical config shape is `ai_sources/core/config/sources.json` (schema_version 1, `timezone`, `max_workers`, `defaults`, `sources` map of source_id → adapter config). There is NO source-set abstraction; a "per-window config" is just a sources.json-shaped file selecting the chosen sources.
- Source ids confirmed present in `ai_sources/core/config/sources.json` — do NOT invent ids. Categories grouped:
  - 08 urgency-leaning candidates (official_lab + ai_hardware_platform + community_signal + product_launches + industry_news): `openai-blog`, `anthropic-news`, `google-ai-blog`, `google-deepmind-blog`, `meta-ai-blog`, `mistral-news`, `nvidia-blog`, `hacker-news`, `product-hunt`, `techcrunch-ai`, `venturebeat-ai`, `ai-news`. (HN belongs here per T2.)
  - 12 opinion-leaning (expert_analysis + research_commentary + editorial_practice + chinese_business + global_ai_news + newsletter): `sebastian-raschka`, `thegradient`, `towards-ai`, `towardsdatascience`, `36kr`, `synced`, `rundown-ai`.
  - 18 deep/practitioner-leaning (developer_signal + research_preprint + agent_framework + model_infra + developer_platform + cloud_platform + technical_news + tutorials): `github-trending`, `arxiv-ai`, `langchain-blog`, `huggingface-blog`, `groq-blog`, `fireworks-blog`, `cerebras-blog`, `together-blog`, `modal-blog`, `replicate-blog`, `runpod-blog`, `aws-ml-blog`, `microsoft-ai-blog`, `marktechpost-ai`, `analyticsvidhya`, `machinelearningmastery`. (Executor decides final exact picks; this is a defensible candidate pool.)
- README's example filenames suggest naming `configs/sources_08_morning.json`, etc., but the user explicitly said "config目录" (singular config dir). Resolve as `/home/ubuntu/agents/builders/ai-informer-builder/config/window_08.json` (and `_12`, `_18`) — clean per-window names, single dir, mirrors the design vocabulary.
- Forbidden tokens that MUST grep to 0 against the trimmed design after T1/T2: `rubric weights`, `threshold`, `max_shortlist_size`, `depth_budget`, `framing prefix`, `今晨重点`, `中午观点`, `晚间技术`, `reverse-lookup`, `reverse lookup`, `amplifier`. The §3 wake-up step, §3 screener-call step, §3 researcher-call step, §4.1 task packet line, §4.2 task packet line, §5 per-window skill description, and §6 HN amplifier bullet all require rewording — NOT just deletion. The screener still receives a packet; it carries semantic guidance (window theme, relevance criteria as natural-language hints, prior dedupe keys), not numerical knobs.
- Hard constraints carried forward: design ≤100 lines (target band 60–90 after deletions); Feishu webhook URL stays inline at §6 only — must NOT appear in any new config/outline file; no ai_sources/ mutation; no scaffolding (no AGENTS.md authoring outside Builder's CLAUDE.md, no .codex/agents/*.toml, no .agents/skills/, no ai_informer/, no copy.sh); no cron entries; no pre-implementing screener/researcher in code.
- Per-window skill outlines: each ≤40 lines, 4-stage structure (a) collection instruction (which sources_config and what `since` value), (b) screener semantic guidance (window theme + relevance criteria in natural language), (c) researcher focus per shortlisted candidate (what to verify/explain), (d) final output writing (already-fixed shape: ≤200 字 summary + per-item Title / Why-worth-reading / URL, Chinese, no markdown).
- Subagent outlines: each ≤50 lines. Semantic-prompt outlines, NOT .toml. Each covers role/responsibility, inputs received, semantic guidance for behavior, return shape, hard prohibitions (no file writes, no message I/O for screener; researcher web-read within policy only).
- Containment: webhook URL stays only in design line ~68; configs and outlines must not duplicate it.
- Mailbox reply: Chinese, no markdown, via `.claude/skills/mailbox-operate/scripts/send_mailbox.py --to human --await-reply`. Confirm trim applied, list pointers to 8 new files, ask for approval before scaffolding (t2 subtasks 2+).
- Todos: t2 stays pending — this episode is detail design, not scaffolding. No subtask transitions expected unless a new subtask gets introduced for "detail design produced". t1, t3, t5 already done; t4, t6, t7 untouched (post-implementation).
- Knowledge: no new note this round unless executor finds a genuinely new structural insight; the rewrite-under-line-cap pattern is still below the 3-occurrence threshold.

## Actions Taken

- Read the updated `ai_sources/README.md`, `ai_sources/core/cli.py`, and `ai_sources/core/config/sources.json`. Confirmed CLI is `(sources_config_path, today|week|month)` and the canonical source-id list lives in `sources.json`.
- Trimmed `design/ai_informer_agent_design.md` per T1 + T2: removed all numerical-knob tokens; reshaped §4.1 / §4.2 task packets into pure semantic guidance + one-line semantic fetch constraint; rewrote §3 steps 4–5 to drop knobs; rewrote §5 to describe per-window skills as carriers of semantic guidance + source-config pointer; replaced the §6 HN reverse-lookup amplifier bullet with a "HN engagement metrics in `raw_items[*].source_metrics` available as semantic context for 08" bullet. Added the reference line pointing at `config/window_*.json`, `design/skills/window-*.md`, `design/subagents/{screener,researcher}.md`.
- Created `config/window_08.json` (12 sources, `since=today`), `config/window_12.json` (7 sources, `since=today`), `config/window_18.json` (14 sources, `since=week`, `defaults.limit=15`). Each id verified against `ai_sources/core/config/sources.json`; none invented.
- Created `design/skills/window-08-urgency.md`, `design/skills/window-12-opinion.md`, `design/skills/window-18-deep.md` — each ≤40 lines, 4-stage outline (collection / screener semantic guidance / researcher focus / final two-part Chinese writing).
- Created `design/subagents/screener.md` (44 lines) and `design/subagents/researcher.md` (47 lines) — single responsibility, input shape, behavior guidance, return shape, side-effect boundaries.
- Sent mailbox reply via `send_mailbox.py --to human --await-reply`, Chinese, no markdown, listing all 8 new artifact paths and asking for approval before scaffolding.

## Key Evidence

- `wc -l design/ai_informer_agent_design.md` → 70 (target band 60–90; ≤100 cap).
- Skill outlines line counts: 35 / 34 / 34 (cap 40). Subagent outlines: 44 / 47 (cap 50).
- Forbidden-token grep `rubric weights|threshold|max_shortlist_size|depth_budget|framing prefix|今晨重点|中午观点|晚间技术|reverse-lookup|reverse lookup|amplifier` (case-insensitive) over the trimmed design AND all 8 new artifacts → 0 matches.
- Webhook URL containment: `open.feishu.cn` appears exactly once in `design/ai_informer_agent_design.md` (line 69) and 0 times across `config/window_*.json`, `design/skills/window-*.md`, `design/subagents/*.md`.
- All three configs parse as JSON; all source ids subset-of `ai_sources/core/config/sources.json` (`missing-from-canonical: set()` for all three).
- Mailbox tail entry id captured below in execution report.

## Outcome

## Outcome

PASS round 1. Evaluator independently verified all ten spot-checks. Trimmed design is 70 lines (≤100, in 60–90 band). Forbidden-token grep returns 0 across the trimmed design AND each of the 8 new files for all 11 banned tokens (rubric weights / threshold / max_shortlist_size / depth_budget / framing prefix / 今晨重点 / 中午观点 / 晚间技术 / reverse-lookup / reverse lookup / amplifier). Webhook URL containment correct: line 69 in design exactly once, 0 in any new file. §5 reference line at line 61 points at config/ + design/skills/ + design/subagents/. All 3 configs parse as JSON; every source id verified subset of ai_sources/core/config/sources.json (37 ids canonical; 12+7+14=33 picks, none missing). Skill outlines 35/34/34 lines (≤40); subagent outlines 44/47 lines (≤50); each carries the prescribed structure. Mail mail.20260505T133321Z.001 from AI Informer Builder to human, await_reply=true, Chinese prose with no markdown markers, names all 8 file paths. No forbidden side effects (no AGENTS.md outside Builder's CLAUDE.md, no .codex/agents/, no .agents/skills/, no ai_informer/, ai_sources untouched). Semantic reshaping in §4.1 is genuine — natural-language theme + plain-hint relevance criteria + prior_recommended_dedupe_keys + recency intuition + now_utc; "Quality over volume; no fixed cap"; the "~12 hours / ~7 days" recency wording is illustrative natural-language hint, not a tunable knob.

## Reflection

The user's two trim instructions removed the surface area for a whole class of mistake — treating LLM-based screening as a knob-tunable recommender. Reshaping §4.1 to "natural-language theme + plain-hint relevance criteria + recency intuition" is the intended design language; numerical knobs are out. The HN amplifier deletion settled cleanly because Q3 was already framed as DEFAULT not BLOCKING; HN becomes one of twelve sources in window_08 with engagement metrics available in raw_items[*].source_metrics as semantic context. The bigger structural shift this episode landed is the move to a layered design artifact: the main design doc (70 lines) carries the contract; per-window source-set details live in config/window_*.json (referenced, not inlined); per-window business logic lives in design/skills/window-*.md as 4-stage outlines (collection / screener-guidance / researcher-focus / output-writing); subagent semantic-prompt outlines live in design/subagents/{screener,researcher}.md. This layering keeps the main design under cap while giving t2 enough concrete content to scaffold without rediscovery. The evaluator's heuristic-candidate observation about knobs sneaking back as illustrative examples is real and worth keeping in mind — the trim discipline is "an inline natural-language example is fine; a named parameter or bracketed knob is not"; one more occurrence and a heuristic note becomes justified.

## Follow-up Actions

- Did NOT mark t2 done — still pending human approval on mail.20260505T133321Z.001 (await_reply=true; runtime gates the next heartbeat on this).
- t2's existing subtask 1 was already done last heartbeat with the Feishu resolution; no t2 subtask transitions needed this episode. Subtasks 2-8 remain pending; will become live on human approval.
- Did NOT promote a new knowledge note. The "knobs sneaking back as illustrative examples" heuristic is one occurrence so far; the existing heuristic--codex-agent-design--audit-builder-harness-vocabulary is in spirit a similar trim-discipline note; defer until a second occurrence justifies a dedicated note.
- The mailbox send used --await-reply (second use of the rule). The next heartbeat is gated on the human's reply (approval / further changes / drift to scaffolding).
- No t4/t6/t7 changes (implementation-phase adapter-health items remain untouched).
