---
doc_id: implement_plan.ai-informer
status: draft
last_edited_at: 2026-05-06
---

# AI Informer — Implementation Plan

## Goal

Build AI Informer as a vertical Codex agent on top of the harness `basic-agent` scaffold, following the layering taught by `.claude/skills/build-a-codex-agent/SKILL.md`. This file is the contract for t2 subtasks 2-9; it does not duplicate design content but cites sections by path. Discipline: smoke-gate the scaffold immediately after copy, then customize in fixed order, then smoke-gate end-to-end before installing any cron entry.

## Prerequisites and required reading

Read once before starting any step:

- `.claude/skills/build-a-codex-agent/SKILL.md` — mental model (main agent vs skills vs subagents), AGENTS.md guidance, when to add a skill, when to add a Codex subagent. Authoritative for layering decisions.
- `.claude/skills/build-a-codex-agent/scripts/copy.sh` — the helper that seeds `basic-agent` into the workdir; CLI shape is `bash <skill>/scripts/copy.sh --dest <abs-path> [--name <slug>] [--no-install]`.
- `design/ai_informer_agent_design.md` — §1 business goal; §2 architecture; §3 runtime mechanism (the four file-mediated stages); §4 subagent contracts; §5 per-window skills; §6 data invariants and `Runtime/runs/<window>/<YYYYMMDD-HHMM>/` artifact layout.
- `design/notifications.md` — module path `ai_informer/notifications/`, CLI shape, registry-style adapter discovery by `type`, `config/notifications.json` as single source of truth.
- `design/skills/window-08-urgency.md` — Stage 1 `today`, urgency relevance guidance, researcher focus.
- `design/skills/window-12-opinion.md` — Stage 1 `today`, opinion relevance guidance, researcher focus.
- `design/skills/window-18-deep.md` — Stage 1 `week`, practitioner relevance guidance, researcher focus, `defaults.limit=15`.
- `design/subagents/screener.md` — single responsibility, file-mediated I/O, `stripped.txt` line format, return contract.
- `design/subagents/researcher.md` — per-record prompt-borne input, bounded web reads, natural-language briefing return.

## Starting point — what `scaffolds/basic-agent` provides

Source path: `/home/ubuntu/agents/long-run-agent-harness/engine/scaffolds/basic-agent` (read-only). `copy.sh` cp -r's it into the workdir, sets `package.json:name`, then `npm install && npm run build` unless `--no-install` is passed.

Top-level shape produced in the workdir:

- `AGENTS.md` — placeholder system prompt; will be replaced (step 2).
- `CLAUDE.md` — Claude-Code convention; not authoritative for a Codex agent. Decide retain/adjust/remove in step 2.
- `package.json`, `package-lock.json`, `tsconfig.json` — Node/TypeScript project; do not edit beyond `name`.
- `src/runtime/{client.ts,run.ts}` and `src/trajectory/{schema.ts,recorder.ts}` — runtime contract; per SKILL.md leave alone unless intentionally changing the harness.
- `src/entry/{one-shot.ts,wake-up.ts,scheduled.ts}` — entry points the cron and one-shot runs target.
- `src/loaders/{skills.ts,system-prompt.ts}` — auto-discovery of `.agents/skills/` and `AGENTS.md`.
- `scripts/run-once.sh`, `scripts/install-systemd-timer.sh`, `scripts/cron.example` — operational scripts; cron entries are appended in step 12.
- `probes/probe-stream.mjs`, `probes/probe-skills.mjs` — smoke probes used by the step 1 gate.
- `.codex/agents/web_researcher.toml.example`, `.codex/agents/README.md` — example subagent config; real subagents are authored in steps 3-4.
- `.gitignore` — standard.

## Target directory layout (post-scaffold, AI-Informer-specific files only)

```
<workdir>/
  AGENTS.md                                     # replaced; from design §2 + §3
  .codex/agents/screener.toml                   # from design/subagents/screener.md
  .codex/agents/researcher.toml                 # from design/subagents/researcher.md
  .agents/skills/
    window-08-urgency/SKILL.md                  # from design/skills/window-08-urgency.md
    window-12-opinion/SKILL.md                  # from design/skills/window-12-opinion.md
    window-18-deep/SKILL.md                     # from design/skills/window-18-deep.md
    _shared/scripts/strip_for_screener.py       # shared helper; see design §3 + screener.md
  ai_informer/                                  # Python package, parallel to ai_sources/
    __init__.py
    notifications/
      __init__.py
      cli.py                                    # entrypoint per design/notifications.md
      registry.py                               # type→adapter resolver (registry-style)
      channels/
        __init__.py
        feishu.py                               # first adapter
  config/
    notifications.json                          # SINGLE source of truth for webhook URLs
    window_08.json  window_12.json  window_18.json   # already authored
  Runtime/runs/<window>/<YYYYMMDD-HHMM>/        # AI Informer per-run artifact root
    collected.json  stripped.txt  shortlist.json
    briefings/<record_id>.md
    output.md
  scripts/cron.example                          # entries appended for 08/12/18 (NOT installed)
  README.md                                     # updated to point at design/ + this plan
```

The `Runtime/` root above is the AI Informer runtime artifact tree. It is distinct from the BUILDER agent's `Runtime/agent.json` runtime state. The earlier draft's fourth rendering skill is retired; only the three window skills plus the shared helper exist.

## Build sequence

Each step lists input, action, done-when, and risk. Proceed in order. If any step's done-when fails, stop and report.

1. Seed scaffold.
   - input: `.claude/skills/build-a-codex-agent/SKILL.md`, `copy.sh`.
   - action: stage-then-move — run `bash .claude/skills/build-a-codex-agent/scripts/copy.sh --dest /tmp/ai-informer-staging --name ai-informer-builder` (the helper resolves the scaffold under the harness `engine/scaffolds/basic-agent` and runs `npm install` + `tsc` inside the staging dir), then `cd /tmp/ai-informer-staging && node probes/probe-stream.mjs`; on verdict pass, `mv` the scaffold artifacts (AGENTS.md, .codex, .gitignore, package.json, package-lock.json, probes, scripts, src, tsconfig.json, node_modules, dist) into the workdir excluding scaffold's `CLAUDE.md` and re-run `node probes/probe-stream.mjs` from the workdir to confirm. (See ep.20260506T030000Z for the empirically validated invocation pattern; copying directly into a non-empty workdir is unsafe.)
   - done-when: probe verdict `pass` exit 0 in both staging and workdir; the workdir top-level matches the Starting-point shape; Builder `CLAUDE.md` sha256 unchanged.
   - risk: probe failure → stop; do not continue to step 2 with a broken scaffold.

2. Replace `AGENTS.md` (and decide on `CLAUDE.md`).
   - input: `design/ai_informer_agent_design.md` §2 + §3.
   - action: write the AI Informer business-specific system prompt into `AGENTS.md` at workdir root; assess whether the scaffold's `CLAUDE.md` should be retained, adjusted, or removed (Codex agent authoritative prompt is `AGENTS.md`, not `CLAUDE.md`).
   - done-when: spot-check by waking the Codex agent with a no-op prompt — identity and three-layer model are correctly reflected.
   - risk: putting business judgment into a skill instead of `AGENTS.md` (per SKILL.md common mistakes).

3. Author `.codex/agents/screener.toml`.
   - input: `design/subagents/screener.md`.
   - action: encode single responsibility, expected task packet (paths + window guidance + prior dedupe keys), file-read/write boundary, and the natural-language return contract.
   - done-when: invoking the screener with a hand-crafted task packet produces a `shortlist.json` and a filtering summary.
   - risk: granting web fetch — explicitly disallowed by the contract.

4. Author `.codex/agents/researcher.toml`.
   - input: `design/subagents/researcher.md`.
   - action: encode per-record prompt-borne input, bounded web-read boundary, natural-language briefing return.
   - done-when: invoking the researcher on a hand-crafted record returns a substantive briefing with evidence pointers.
   - risk: granting file writes — disallowed; researcher is read-only.

5. Author `.agents/skills/_shared/scripts/strip_for_screener.py`.
   - input: `design/ai_informer_agent_design.md` §3, `design/subagents/screener.md` (line-format spec).
   - action: implement `collected.json` → `stripped.txt`; per-line format `<title> | <sources> [<source_categories>]: <summary excerpt> | <published_at> | id=<record_id>`; `raw_items` excluded; multi-source values joined.
   - done-when: running on a real `collected.json` from a `python -m ai_sources.core.cli` run produces a `stripped.txt` whose lines match the spec.
   - risk: leaking `raw_items` or omitting `record_id` — both break the screener contract.

6. Author `.agents/skills/window-08-urgency/SKILL.md`.
   - input: `design/skills/window-08-urgency.md`.
   - action: encode the four stages — Stage 1 `today` collection, Stage 2 helper + screener with urgency guidance, Stage 3 researcher focus, Stage 4 delivery via notifications CLI.
   - done-when: stage 1 + stage 2 dry-run on real data produces `collected.json` then `stripped.txt` then `shortlist.json`.
   - risk: encoding window guidance into the subagent files rather than the skill.

7. Author `.agents/skills/window-12-opinion/SKILL.md`.
   - input: `design/skills/window-12-opinion.md`.
   - action: same shape as step 6; `today`; opinion relevance guidance.
   - done-when: same as step 6.
   - risk: same as step 6.

8. Author `.agents/skills/window-18-deep/SKILL.md`.
   - input: `design/skills/window-18-deep.md`.
   - action: same shape; `week`; practitioner guidance; remember `defaults.limit=15`.
   - done-when: same as step 6, with attention to `github_trending` HTML-in-summary and null `published_at`.
   - risk: same as step 6.

9. Author `ai_informer/notifications/`.
   - input: `design/notifications.md`.
   - action: implement `cli.py` (`--message-file <path> [--channel <name>]`, broadcast by default), a registry resolver keyed on the config `type` field, and `channels/feishu.py` exposing `send(message)`.
   - done-when: `python -m ai_informer.notifications.cli --message-file <fixture>` against a test fixture sends to a configured channel; unit test on `feishu.send()` passes.
   - risk: hard-coding the webhook URL in code — it must live only in `config/notifications.json`.

10. Create `config/notifications.json`.
    - input: `design/notifications.md` (config shape).
    - action: write the file with one channel entry of `type: "feishu"` plus the configured webhook URL. This is the only file in the repo that contains the literal URL.
    - done-when: file exists; CLI default broadcast resolves and dispatches to the Feishu adapter.
    - risk: pasting the URL elsewhere (skills, design files, code).

11. End-to-end smoke run for one window (manual, no cron yet).
    - input: a chosen window (recommend 08 or 12 since `today` produces faster batches).
    - action: drive Stages 1-4 by hand; verify the artifact chain `collected.json → stripped.txt → shortlist.json → briefings/<record_id>.md → output.md`, then invoke the notifications CLI and confirm Feishu receipt.
    - done-when: every artifact is present, `output.md` is Chinese no-markdown with a ≤200 字 summary and per-item Title/Why-worth-reading/URL, Feishu delivery confirmed.
    - risk: any missing artifact or delivery failure → stop and fix before step 12.

12. Document cron entries in `scripts/cron.example` for 08:00 / 12:00 / 18:00.
    - input: existing scaffold `scripts/cron.example` and `scripts/install-systemd-timer.sh`.
    - action: append three commented entries; do NOT install. Installation only after step 11 passes and the human approves.
    - done-when: file diff shows three entries; nothing is registered with the OS.
    - risk: installing cron prematurely — explicit stop-and-report condition.

## Verification strategy

Step 1: probe completes 0. Step 2: no-op wake produces correct identity. Steps 3-4: hand-crafted task packets exercise each subagent end-to-end. Step 5: helper output spot-checked against the line-format spec. Steps 6-8: stage 1 + stage 2 dry-run yields `shortlist.json`. Step 9: unit test on `feishu.send()` plus a CLI dry-run against a test channel. Step 10: CLI default broadcast resolves the channel by `type`. Step 11 is the full smoke gate before any cron work. Step 12: cron is documented but not installed.

## Cron installation

Deferred until step 11 passes. The source of truth for cron entries is `scripts/cron.example`. Installation goes via `scripts/install-systemd-timer.sh` (shipped by the scaffold) or via `crontab -e`; either way installation is the final, human-gated act, not part of authoring.

## Out of scope

`ai_sources/` adapter health repairs are deferred until `ai_sources` mutation is permitted: t4 (github_trending stars=0), t6 (meta-ai-blog HTTP 400), t7 (reddit-ai HTTP 403). Not in this build sequence. The reference impl at https://github.com/Tom-0727/AInformer/blob/main/core/utils/inform.py informs the notifications design but is not copied verbatim — see `design/notifications.md` for the deliberate divergence (config-file plus named channels rather than env-var plus URL-substring platform sniffing).
