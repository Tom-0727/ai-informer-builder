---
id: ep.20260507T022806Z.knowledge.promote-three-medium-priority-build-notes
task_id: task.knowledge.promote-build-heuristics
domain: knowledge
title: Promote three medium-priority build notes (foreground-validate, filesystem-marker, ThreadOptions schema)
objective: Three new knowledge notes land — two heuristic notes under Memory/knowledge/heuristic/ (foreground-validate-before-scheduled-trigger; satisfy-marker-shape over deep engine edits) and one factual note under Memory/knowledge/factual/ (Codex SDK ThreadOptions schema fields) — each conforming to Memory/knowledge/README.md, matching the dominant existing heuristic-note cadence (32-42 lines), with at least one episode-id provenance, no literal Feishu webhook URL or token form, and Builder CLAUDE.md sha256 unchanged.
status: completed
eval_rounds: 1
last_edited_at: 2026-05-07
---

## Objective

Author three new knowledge notes that distill three medium-priority insights accumulated during the AI Informer build correction trajectory: the foreground-validate-before-scheduled-trigger discipline, the filesystem-marker-satisfaction-over-deep-engine-edits heuristic, and the Codex SDK ThreadOptions schema reference. The episode is "done" when all three notes exist with canonical frontmatter and the matching body sections, each references at least one source episode, contains no literal webhook URL or token form, and the Builder workspace's other tracked files are untouched. Notes are working-tree only; no commit this heartbeat.

## Context Snapshot

- Drives the t16 medium-priority knowledge promotion thread. t14 (engine scaffold disposition), t15 (first cron run verification), t17 (gated on human direction), t4/t6/t7 (post-implementation, deferred) all explicitly out of scope.
- Build is in steady-state production-grade: cron entries installed today at ~09:12 local Beijing UTC+8; today's 08:00 fire was BEFORE cron install so no cron output yet; next fire window is 12:00 local. t15 has no data yet.
- Prior knowledge promotion episode ep.20260507T013930Z PASSED round 1 and authored three high-priority heuristic notes. This episode picks up the next three from the same promotion queue, deliberately limited to standalone-note candidates; skill-shaped candidates (#6 structural verification gates checklist, #7 Codex agent artifact authoring contract) and narrow single-instance candidates (#4 PATH B smoke checklist, #5 monkeypatch urlopen seam) are deferred.
- Workspace conventions confirmed via README and existing exemplars: `<kind>--<domain>--<slug>.md` naming, frontmatter (id/kind/summary/last_edited_at/status/tags), four-section body (Core Content / Boundary Conditions / Limitations / Optional Notes for heuristic; Core Content / Limitations / Optional Notes for factual). Existing heuristic-note line cadence is 32-42 lines.
- Builder CLAUDE.md sha256 baseline before this episode: 9a0dfbdd819368df248e777d85a7b7a58041df980cb1d2cbb5edee107572a632.
- Source episodes for provenance:
  - NOTE 1 foreground-validate: ep.20260506T060000Z (PATH A foreground validation surfaced sandbox DNS issue in 9 minutes vs hypothetical cron-driven debugging).
  - NOTE 2 filesystem-marker satisfaction: ep.20260506T083000Z and ep.20260506T143000Z (empty `.git/` anchor for Codex trusted-directory walk).
  - NOTE 3 ThreadOptions schema: ep.20260506T050000Z (subagent .toml authoring — initial discovery), ep.20260506T230500Z (sandbox-network coupling discovery confirmed schema).
- Hard constraint: do NOT type the literal Feishu webhook URL form `open.feishu.cn/open-apis/bot/v2/hook/<token>` or the bare token in any new note. None of the three notes reference the webhook URL by topic; verification still runs as a defensive gate.
- Hard constraint: do NOT git add / commit / push. Do NOT flip any Todo (knowledge promotion is observational; t16 has no "promote 3 notes" subtask). No file modifications beyond the three new notes and this episode file. Mailbox progress without --await-reply.
- Stop conditions: reaching for any git mutating operation → STOP; reaching for any Todo flip → STOP; reaching for any other-file modification → STOP; scope expands beyond 3 notes → STOP.

## Actions Taken

- Pre-flight: confirmed Builder CLAUDE.md sha256 baseline 9a0dfbdd...2632; listed `Memory/knowledge/{heuristic,factual}/` and verified zero filename collisions for the three planned slugs.
- Read `Memory/knowledge/README.md` for canonical frontmatter and body template; read the most recent heuristic exemplar `heuristic--codex-sdk--sandbox-network-requires-explicit-mode.md` (37 lines, four sections) and the factual exemplar `factual--ai-sources--adapter-contract-v1.md` (53 lines, three sections — no Boundary Conditions).
- Authored NOTE 1 `heuristic/heuristic--scheduler-deployment--foreground-validate-before-install.md` (32 lines) — OPTION X discipline, scheduled-trigger trap, foreground-cost-vs-recovery-cost asymmetry, generalization to systemd timers / GHA / k8s CronJobs / cloud schedulers; cross-references kn.codex-sdk.heuristic.sandbox-network-requires-explicit-mode.
- Authored NOTE 2 `heuristic/heuristic--third-party-integration--satisfy-marker-shape-over-engine-edits.md` (33 lines) — failure mode → cheapest-first mitigation hierarchy → distinguishability via `git status` → boundary between shape-based and behavior-based heuristic checks; provenance to ep.20260506T083000Z + ep.20260506T143000Z.
- Authored NOTE 3 `factual/factual--codex-sdk--threadoptions-schema.md` (44 lines) — six top-level keys with type/effect/CLI mapping; notable absences (no `web_fetch`, no `tools` allowlist, no global `network_access`); explicit scoped-mapping for `networkAccessEnabled`; matched the factual-exemplar three-section shape (no Boundary Conditions).
- Single-burst defensive secret-pattern grep (full Feishu webhook URL form + bare token) returned 0 matches across all three new notes.
- Sent mailbox progress mail.20260507T023343Z.001 to human (Chinese, no markdown, await_reply false) summarizing the three promotions, line counts, deferred t16 queue residuals, and the working-tree-only status.

## Key Evidence

- `wc -l`: NOTE 1 = 32, NOTE 2 = 33, NOTE 3 = 44 (heuristic in 32-42 cadence; factual ≤ 53).
- Frontmatter verified on each: `id` follows `kn.<domain>.<kind>.<slug>`, `kind` matches subdirectory, single-line `summary`, `last_edited_at: 2026-05-07`, `status: active`, tags array present.
- Section headings: NOTE 1 + NOTE 2 each have `Core Content / Boundary Conditions / Limitations / Optional Notes`; NOTE 3 has `Core Content / Limitations / Optional Notes` only.
- Each Optional Notes section references at least one episode id (NOTE 1 → ep.20260506T060000Z; NOTE 2 → ep.20260506T083000Z + ep.20260506T143000Z; NOTE 3 → ep.20260506T050000Z + ep.20260506T230500Z).
- Defensive secret-pattern grep: 0 / 0 / 0 for the full URL form; 0 / 0 / 0 for the bare token.
- Builder CLAUDE.md sha256 unchanged at 9a0dfbdd...2632 after authoring.
- `git status --short` deltas attributable to this episode: three new untracked notes plus this episode file. All other entries (Runtime drift, prior unrelated untracked notes, prior modified files) predated this episode.
- Outgoing mailbox tail-1: `mail.20260507T023343Z.001`, from `AI Informer Builder`, to `human`, `await_reply: false`.

## Outcome

PASS round 1. Evaluator independently verified all 13 spot-checks: three new files exist at the expected paths under Memory/knowledge/{heuristic,factual}/; line counts 32/33/44 within heuristic 32-42 cadence and within factual ≤53 cap; frontmatter on each conforms to README convention (id with kn.<domain>.<kind>.<slug> pattern, kind matching subdirectory, single-line summary, last_edited_at: 2026-05-07, status: active, tags as flat list); body shape matches dominant existing style (NOTE 1+2 carry Core Content + Boundary Conditions + Limitations + Optional Notes; NOTE 3 deliberately omits Boundary Conditions per the factual exemplar shape); Optional Notes provenance present on all three with episode-id citations; cross-references to kn.codex-sdk.heuristic.sandbox-network-requires-explicit-mode resolve cleanly; defensive secret-pattern grep returns zero matches across the entire Memory/knowledge tree for both the literal Feishu URL form and the bare token form; Builder CLAUDE.md sha256 unchanged at canonical baseline; git status --short shows the three new untracked notes + new episode file as expected with no other tracked-file mutations; ai-informer/ HEAD remains at deccf33 (no commits made there); mailbox tail confirms Chinese, no markdown markers, await_reply=false. Quality spot-checks confirmed each note is direct and decision-relevant; voice pairs concrete trigger + generalization + boundary; the schema/heuristic cross-reference forms a coherent hub-and-spoke between fact and operational consequence; NOTE 3's "exactly six" claim is internally consistent with the camelCase networkAccessEnabled coexistence properly flagged via the --config sandbox_workspace_write.network_access=true mapping.

## Reflection

Three calls worth recording: (1) The kind-specific section pattern distinction — heuristic notes use four sections (Core Content + Boundary Conditions + Limitations + Optional Notes), factual notes use three (Core Content + Limitations + Optional Notes only) — was correctly applied per the dominant existing style. The planner's pre-validation that the factual exemplar omits Boundary Conditions saved a style-drift bug. The lesson generalizes: when authoring against a workspace's existing knowledge base, always sample multiple existing exemplars per kind to detect kind-specific cadence/section differences, not just one exemplar. (2) Triage discipline (standalone-note candidates vs skill-shaped candidates vs single-instance narrow candidates) proved valuable in scope-bounding: this episode promoted 3 of 7 candidates and correctly deferred the 2 skill-shaped (#6, #7) for a dedicated skill-creation episode and the 2 narrow-instance (#4, #5) until second occurrence. The metacognitive observation: "skill-shaped" vs "knowledge-note-shaped" is a real distinction worth flagging at planning time — skills package executable workflows, knowledge notes capture decision rules. (3) The hub-and-spoke pattern between NOTE 1 + NOTE 3 cross-referencing the prior sandbox-network heuristic forms a coherent retrieval graph where a future reader can land on any one note and traverse to the operational consequence (the heuristic note), the schema (the factual note), and the deployment discipline (the scheduler heuristic). Worth keeping in mind for future knowledge promotion passes — explicit cross-references make the knowledge base navigable.

## Follow-up Actions

- Episode `status: completed`, `eval_rounds: 1`. Done.
- Three notes are working-tree-only — pending a future Builder commit episode that bundles them with: (a) the prior 3 high-priority notes from ep.20260507T013930Z (also working-tree-only); (b) RETROSPECTIVE_v1.md from ep.20260507T020000Z (also working-tree-only); (c) any t15 first-cron-run observations. The next commit episode should sweep all of these together with a fresh literal-form grep gate per the heuristic--secret-redaction discipline.
- t16 still has 4 candidates queued: 2 skill-shaped (#6 structural verification gates checklist, 4-instance; #7 Codex agent artifact authoring contract, 3-instance) — defer to a dedicated `.claude/skills/<name>/SKILL.md` skill-creation episode; 2 single-instance narrow (#4 PATH B four-stage smoke checklist; #5 monkeypatch urlopen seam test pattern) — defer until second occurrence.
- t14 (engine scaffold disposition), t15 (first cron run verification), t17 (retrospective slimming-list routing) remain pending — all gated on either human direction or scheduled fire.
- t4, t6, t7 (adapter health) remain post-implementation deferred.
- Distillation candidate from this housekeeping pass: a "knowledge-promotion-checklist" sub-procedure (read README + sample multiple kind-exemplars + author N notes with bound-shell-variable secret guards + run literal-form grep gate + mailbox progress) has now run cleanly twice in succession. If a third occurrence comes up (e.g., the deferred skill-creation episode for #6 + #7), this template would crystallize into a small reusable skill — but it's still below promotion threshold at two instances.
- The next heartbeat will likely activate when (a) human reviews the recent retrospective + knowledge promotion + provides routing for t17 + t14, OR (b) first cron fires at 12:00 local today (~1.5 hours from now) producing t15 data.
