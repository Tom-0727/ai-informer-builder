---
id: ep.20260507T013930Z.knowledge.promote-three-high-priority-build-heuristics
task_id: task.knowledge.promote-build-heuristics
domain: knowledge
title: Promote three high-priority build heuristics into Memory/knowledge/
objective: Three new heuristic notes land in Memory/knowledge/heuristic/ — Codex SDK sandbox network coupling, single-burst secret redaction in heartbeat-logged workspaces, and long-running-agent secret accumulation — each conforming to Memory/knowledge/README.md and matching the existing heuristic-note style, with episode-id provenance, no literal Feishu webhook URL or token, and Builder CLAUDE.md sha256 unchanged.
status: completed
eval_rounds: 1
last_edited_at: 2026-05-07
---

## Objective

Author three new heuristic knowledge notes under Memory/knowledge/heuristic/ that distill the three highest-priority insights accumulated during the AI Informer build. The episode is "done" when each note exists with canonical frontmatter and body sections, references at least one source episode, contains no literal webhook URL or token, and the Builder workspace's other tracked files are untouched. Notes are working-tree only; no commit this heartbeat.

## Context Snapshot

- Drives no Todo directly. t14 (engine scaffold disposition) and t15 (first cron run verification) remain pending and out of scope.
- Build is in steady-state production-grade: cron installed, both repos pushed, awaiting first cron fire.
- Workspace conventions confirmed: Memory/knowledge/README.md prescribes `<kind>--<domain>--<slug>.md` naming, frontmatter with id/kind/summary/last_edited_at/status/tags, and a Core Content / Limitations / Optional Notes body. Existing heuristic notes (`heuristic--codex-agent-design--audit-builder-harness-vocabulary.md`, `heuristic--scaffold-adoption--vendored-sdk-binary-system-upgrade-no-help.md`) add a "Boundary Conditions" or "When this heuristic fires" section between Core Content and Limitations — match that cadence.
- Builder CLAUDE.md sha256 baseline before this episode: 9a0dfbdd819368df248e777d85a7b7a58041df980cb1d2cbb5edee107572a632.
- Source episodes for provenance: ep.20260506T060000Z (window-skills + sandbox network first manifestation), ep.20260506T143000Z (cleanup/relocate/revalidate + secrets safety gate), ep.20260506T230500Z (sandbox-escalation + Stage D halt), ep.20260507T011000Z (redact/commit/push/cron-install — exemplar of single-burst redaction discipline).
- Hard constraint: do NOT type the literal Feishu webhook URL form `open.feishu.cn/open-apis/bot/v2/hook/<token>` or the bare token in any new note; describe the pattern abstractly or via shell-variable form. Verification must use a literal-form grep against the staged tree but conducted via a single shell burst so the pattern itself is bound to a shell variable, never echoed as a tool argument.
- Stop conditions: if README convention diverges from plan, adapt to README; if note count would exceed 3, stop; do NOT run any git commit/push.

## Actions Taken

- Pre-flight: confirmed Builder CLAUDE.md sha256 matches baseline 9a0dfbdd819368df248e777d85a7b7a58041df980cb1d2cbb5edee107572a632; listed Memory/knowledge/heuristic/ — only the two prior exemplar notes present, no filename collision.
- Read Memory/knowledge/README.md and the two existing heuristic exemplars to lock the section cadence (Core Content / Boundary Conditions / Limitations / Optional Notes) and the dominant note size (existing notes are 32 and 42 lines).
- Authored three new notes in Memory/knowledge/heuristic/, each with canonical frontmatter (id, kind, summary, last_edited_at: 2026-05-07, status: active, tags), all four required sections, and at least one episode-id reference under Optional Notes.
- Note 1 (Codex SDK sandbox network coupling): captures the SDK-to-CLI flag mapping, the gaierror -3 diagnostic signature, the workspace-write fix, and danger-full-access as broader fallback. Provenance: ep.20260506T060000Z, ep.20260506T143000Z, ep.20260506T230500Z (canonical Tier 1 fix).
- Note 2 (single-burst secret redaction discipline): captures the tool-use-transcript leakback failure mode, the shell-variable-bound pattern, the LAST-write ordering for the heartbeat-logged file, the no-yield bash chain, and folds in the verify-by-literal-form-not-prefix corollary as a Core Content paragraph. Provenance: ep.20260507T011000Z (canonical exemplar) and ep.20260506T230500Z (initial leak discovery).
- Note 3 (long-running-agent secrets architectural framing): captures the propagation observation (one mailbox secret reaching 15+ files), the four-option surface (private+rotation, write-time redaction, batch redaction, out-of-band channel), and the boundary that vertical Codex agents are out of scope. Provenance: ep.20260506T143000Z, ep.20260506T230500Z, ep.20260507T011000Z.
- Verification gate run as a single shell burst with shell-variable-bound patterns (PATTERN_URL via printf composition, PATTERN_TOKEN as 8-hex prefix). Both literal-form greps returned 0 across all three notes.
- Sent Chinese no-markdown progress mailbox via mailbox-operate send_mailbox.py without --await-reply (mail.20260507T014509Z.001).
- No git mutating operation performed; notes remain working-tree only.

## Key Evidence

- Three new files created (each within the dominant existing-style line range, deliberately tighter than the planner's 70-150 ceiling per stop-and-report condition #2):
  - Memory/knowledge/heuristic/heuristic--codex-sdk--sandbox-network-requires-explicit-mode.md (36 lines)
  - Memory/knowledge/heuristic/heuristic--secret-redaction--single-burst-in-heartbeat-logged-workspace.md (38 lines)
  - Memory/knowledge/heuristic/heuristic--long-running-agent-secrets--accumulate-in-records.md (41 lines)
- Section headings on each new note: Core Content, Boundary Conditions, Limitations, Optional Notes.
- Frontmatter on each new note confirmed via head -10: id, kind: heuristic, summary, last_edited_at: 2026-05-07, status: active, tags array present.
- Single-burst verification grep result: 0 matches for the full webhook URL form, 0 matches for the bare token-prefix form, across all three notes.
- Builder CLAUDE.md sha256 post-write: 9a0dfbdd819368df248e777d85a7b7a58041df980cb1d2cbb5edee107572a632 — unchanged from baseline.
- git status --short post-write: three new untracked notes plus the new episode file plus the same Runtime/mailbox/todo drift that was present before this episode (events.jsonl, last_heartbeat, runtime.log, start.log, metrics.json, pending_messages/human.json, mailbox/human.jsonl, todo_list/202605/05.json) plus the M on the prior commit episode that predated this run. No other tracked files mutated.
- Mailbox tail-1: outgoing mail.20260507T014509Z.001 to human, await_reply: false, content in Chinese with no markdown markers.
- Deviation from planner: line counts (36/38/41) are below the planner's 70-150 window but match the dominant existing-style range (32/42 lines for the two prior heuristic notes). Per stop-and-report condition #2 ("Existing heuristic notes use a substantially different style → match dominant existing style"), matching the existing style was prioritized over the planner's line-count window. Notes are content-substantive: each covers all planner-specified Core Content points with bullet-packed phrasing matching the existing exemplars.

## Outcome

PASS round 1. Evaluator independently verified all 15 spot-checks: three new heuristic notes exist alongside the two prior exemplars (5 total in heuristic/ now); each note has full frontmatter (id following kn.<domain>.heuristic.<slug> convention, kind, summary, last_edited_at: 2026-05-07, status: active, tags as list); each has the four section headings (Core Content / Boundary Conditions / Limitations / Optional Notes); each references at least one episode id under Optional Notes (3, 2, 3 references respectively); secret-leak guard via `grep -E 'open\.feishu\.cn/open-apis/bot/v2/hook/[0-9a-f]'` returned zero matches across the three notes + this episode + the mailbox; the residual `open.feishu.cn` mention in the episode body is the documented placeholder pattern `.../<token>` (literal angle-bracket placeholder), not a real token; 8-hex matches in all files are sha256 hashes and timestamps, not webhook tokens; Builder CLAUDE.md sha256 unchanged at `9a0dfbdd...`; no git mutating operation occurred (`git log --oneline` shows only the initial commit `712b2e5`); `git status --short` shows three new untracked notes + new episode + expected Runtime drift; mailbox `mail.20260507T014509Z.001` confirmed Chinese, no markdown markers, await_reply false, mentions all three notes with line counts and deviation rationale; the verify-by-literal-form-not-prefix observation is correctly folded into NOTE 2 line 17 (not a fourth file); quality spot-checks confirmed each note is content-substantive and concrete.

The evaluator's three quality-pass observations — NOTE 1 correctly maps networkAccessEnabled → --config sandbox_workspace_write.network_access=true and explains the workspace-write profile dependency with the gaierror -3 diagnostic; NOTE 2 captures the leak-and-relog failure mode plus the printf-shell-variable binding pattern plus the LAST-write ordering plus the no-yield bash chain; NOTE 3's four architectural options (private+rotation / write-time redaction / batch redaction / out-of-band) are clearly distinguished with trade-offs and a "B and D scale, C is first-pass only, A needs strong privacy invariant" decision frame.

Line count deviation accepted: matching dominant existing style was the right call per planner stop-and-report condition #2.

## Reflection

Three calls worth recording: (1) The knowledge-promotion sub-procedure (read README + existing exemplars → match cadence → author N notes with bound shell-variable secret guards → run literal-form grep gate → mailbox progress) recurred cleanly here. The evaluator flagged this as a medium-signal skill candidate. If the next promotion pass for the remaining 4-7 medium-priority candidates happens, packaging this into a small `knowledge-promote` skill would save effort. Light signal — defer until next pass actually happens. (2) The intentional cross-reference between NOTE 2 ("redaction mechanics") and NOTE 3 ("architectural channel choice") demonstrates a useful pattern: when a heuristic spans both procedural and architectural layers, splitting them into two notes that explicitly reference each other is preferable to forcing one big note. NOTE 2 documents the immediate workaround when the leak has already happened; NOTE 3 documents the structural choice that prevents the leak from recurring. (3) The line-count window deviation (36-41 vs. 70-150) was the right call because the workspace's two prior heuristic notes set the dominant cadence at 32-42 lines. A planning packet's suggested line range is a guideline; the Memory/knowledge/README.md "match existing style" rule supersedes it when there's clear precedent. The evaluator's stop-and-report condition #2 explicitly authorized this and the executor invoked it correctly.

## Follow-up Actions

- Episode `status: completed`, `eval_rounds: 1`. Done.
- Three new notes are working-tree-only — pending a future commit episode. Likely bundled with t15 first cron run observations (per the mailbox note).
- Created new low-priority Todo t16 tracking the medium-priority knowledge promotion queue (4-7 candidates remaining: verify-by-literal-form-not-prefix is folded into NOTE 2; filesystem-marker-satisfaction over deep engine edits; foreground-validate-before-cron-install / OPTION X discipline; Codex SDK ThreadOptions schema fields; PATH B four-stage smoke checklist; monkeypatch urlopen seam test pattern; structural verification gates checklist).
- t14 (engine scaffold disposition) and t15 (first cron run verification) remain pending — explicitly out of scope this episode.
- Roadmap observation: NOTE 3's recommendation that "Option B (write-time redaction at mailbox-send and tool-use-logger) is the durable steady state" is currently an unmet engineering item. Potentially a future Todo "implement write-time secret redaction hook in mailbox-send and tool-use-logger" — but that's an engine-scaffold-level change that requires human direction before action.
- The next commit episode (likely after t15 fires) should bundle: (a) these three new heuristic notes; (b) any t15 observations/artifacts; (c) optional t16 medium-priority promotions if the executor has slack. Run a fresh literal-form grep gate before committing per NOTE 2's discipline.
