# AI Informer — Completion Audit

Generated: 2026-05-06 by ep.20260506T060000Z (PATH A validation episode).
Status: PATH A validation FAILED at the Feishu broadcast stage. Cron install and Todo flips were skipped per the failure protocol. This audit still ships.

## Section A — DONE

Closed Todos:
- t1 — BLOCKING design questions resolved (superseded by ep.20260505T102200Z; design rewrite folded Q1 + Q2). All 3 subtasks done.
- t3 — Multi-source ai_sources.collect validation. PASS r1 via ep.20260505T093400Z. All 3 subtasks done.
- t5 — Magnitude-fallback decision folded into design §6 + §7 with DEFAULT choice. Subtask 1 done; subtask 2 conditional and inert.
- t8 — Engine scaffold SDK pin unblocker. probe-stream verdict pass; basic-agent scaffold seeded successfully. All 4 subtasks done.

t2 subtasks 1-8 done; subtask 9 NOT done (PATH A failed). t2 itself NOT done.
- t2.1 Q1 delivery channel resolved (Feishu webhook).
- t2.2 build-a-codex-agent copy.sh seeded basic-agent into workdir.
- t2.3 AGENTS.md authored from design §2 + §3.
- t2.4 .codex/agents/screener.toml authored.
- t2.5 .codex/agents/researcher.toml authored.
- t2.6 Three window skills (window-08-urgency, window-12-opinion, window-18-deep) plus _shared/scripts/strip_for_screener.py.
- t2.7 Per-run pipeline wired (wake-up → window skill → ai_sources.collect → strip → screener → researcher → synthesis → output).
- t2.8 ai_informer/notifications/ module + config/notifications.json.

Implementation milestones:
- design/ — 8 design files (ai_informer_agent_design.md, ai_informer_design_validation_v{1,2,3}.md, notifications.md, subagents/screener.md, subagents/researcher.md, etc.).
- implement_plan.md (workdir root) — 12-step build manual.
- Scaffold seed via build-a-codex-agent — basic-agent + node_modules + dist/ compiled.
- AGENTS.md (workdir root) — main-agent business prompt.
- .codex/agents/screener.toml + researcher.toml — Codex subagent definitions.
- .agents/skills/_shared/scripts/strip_for_screener.py — shared helper.
- .agents/skills/window-{08-urgency,12-opinion,18-deep}/SKILL.md — three window skills.
- ai_informer/notifications/ Python package — cli.py + registry.py + channels/feishu.py.
- config/notifications.json — broadcast config (single feishu-default channel).
- config/window_{08,12,18}.json — per-window source configs.
- PATH B smoke (window-18, human-verified) — Runtime/runs/18/20260506-0453/ with 6 briefings, output.md, real Feishu broadcast (ep.20260506T044924Z PASS r1, human approved quality and coverage).
- PATH A validation result (this episode, window-08) — autonomous Codex SDK run completed turn.completed, exit 0; produced collected.json (DNS-failed, item_count=0), web_fallback_collected.json (6 records gathered via web_search fallback), web_fallback_stripped.txt (6 lines), shortlist.json (4/6 kept), 4 briefings, output.md (114-char summary, 4 Title/Why/URL triples, zero markdown markers). Notifications CLI invoked but exited 1 with `failed: feishu-default: URLError(gaierror(-3, 'Temporary failure in name resolution'))`. Real Feishu broadcast NOT delivered. Artifacts live at /home/ubuntu/agents/builders/ai-informer-builder/.runs/2026-05-06T05-57-18-224Z-vjfu9a/Runtime/runs/08/20260506-0800/ (Codex sandbox per-run cwd; not in canonical Runtime/runs/08/).

scripts/cron.example NOT modified (still scaffold default). Cron entries NOT installed.

## Section B — PENDING

- t2 subtask 9 — Smoke run for one window without cron (PATH A version). FAILED this attempt at the broadcast stage; failure mode is a Codex sandbox network policy that blocks outbound DNS resolution for the Feishu webhook hostname. The host (outside the sandbox) has full DNS reachability and returns HTTP 404 from the upstream CDN when probed directly, confirming the network path works outside the sandbox. Same DNS isolation also broke the canonical `ai_sources` adapter fetches inside the run; the agent recovered via `web_search` fallback for collection but has no fallback for the Feishu HTTP POST. Until this network-policy gap is resolved, scheduled cron-fired runs will hit the same broadcast failure.
- t2 — itself stays open until t2.9 closes.
- t4 — github_trending stars=0 anomaly. Post-implementation adapter health. Queued.
- t6 — meta-ai-blog HTTP 400. Post-implementation adapter health. Queued.
- t7 — reddit-ai HTTP 403. Post-implementation adapter health. Queued.
- PATH A end-to-end validation — failed this attempt; needs re-run after the broadcast network gap is resolved.

## Section C — NEW WORK SURFACED (audit only, not actioned this heartbeat)

Knowledge promotion candidates (with evidence-trail counts from prior episodes):
- URL containment also covers episode bodies (heuristic candidate; evidence: webhook URL appears in 10 episode files / dozens of occurrences across Memory/episodes; current count ≈53 by line-grep).
- PATH B four-stage smoke checklist (skill candidate, 1 occurrence — ep.20260506T044924Z).
- monkeypatch urlopen seam (skill candidate, 1 occurrence).
- override internal helper for tests (heuristic candidate, 1 occurrence).
- Codex SKILL.md vs Claude-Code frontmatter convention (factual candidate, 2 occurrences).
- Structural verification gates checklist (skill candidate, 4 occurrences).
- Codex subagent TOML schema fields (factual candidate, 2 occurrences).
- Codex agent artifact authoring contract (skill candidate, 3 occurrences).
- SDK-vendored CLI binding (heuristic; already promoted at heuristic--scaffold-adoption--vendored-sdk-binary-system-upgrade-no-help.md).
- NEW from this episode: Codex sandbox outbound DNS isolation breaks naive HTTP-only delivery channels (heuristic candidate). Workspace-write sandbox blocks adapter HTTP and notification HTTP equally; Codex `web_search` is sandbox-allowed, but the notifications CLI uses `urlopen` directly. Mitigation candidates: (a) adjust Codex sandbox network policy at run-once.sh wrap layer, (b) introduce a post-Codex broadcast step in run-once.sh that runs OUTSIDE the Codex sandbox (writes output.md inside, broadcasts outside), (c) move notification dispatch to a Codex-allowed mechanism (e.g., a Codex tool that maps to the host process).
- NEW from this episode: agent autonomous adaptation when canonical adapter fails (heuristic candidate; the agent created `web_fallback_collected.json` and reframed the screener input — this behavior should be documented as an expected failure-mode response rather than an anomaly).
- NEW from this episode: the per-run cwd is the Codex `--cd` workspace (`.runs/<id>/`), not the workdir root, so artifacts land under `.runs/<id>/Runtime/runs/<NN>/<TS>/` rather than `Runtime/runs/<NN>/<TS>/`. PATH B smoke artifacts at `Runtime/runs/18/20260506-0453/` were committed via a different (earlier, possibly direct-run) path. The contract between run-once.sh and the canonical Runtime/runs path needs reconciliation (either Codex `--cd` should be the workdir root, or the cron contract should symlink/move the per-run artifacts back into the canonical tree post-run).

Engine scaffold WT changes:
- /home/ubuntu/agents/long-run-agent-harness/engine/scaffolds/basic-agent/ has uncommitted package.json + package-lock.json edits from t8's SDK pin bump. The audit notes these; the human decides whether to commit upstream, leave local, or revert. Out of scope for this episode.

Historical webhook URL leak count:
- ≈53 occurrences across 10 Memory/episodes/2026/05/*.md files. Cleanup is a follow-up housekeeping episode (separate from production deploy).

implement_plan.md:
- Step 12 (cron.example rewrite + cron install) NOT executed this heartbeat due to PATH A failure. The plan stays accurate; execution gating moves to "after broadcast network gap is resolved".

## Section D — RECOMMENDATIONS

Production state:
- Cron NOT installed; scheduled runs are not firing. Build is functional through synthesis (output.md is well-formed Chinese, 200-char-ceiling summary, no markdown markers, three-field item triples) but the broadcast layer is gated on the Codex sandbox network policy.
- PATH B (manual / human-driven) flow remains operational and was verified end-to-end to real Feishu in the prior episode. PATH B can serve as a temporary delivery mode while PATH A is being fixed.

Recommended next heartbeats (priority order):
1. Diagnose Codex sandbox network policy. Two paths: (a) inspect Codex SDK config for a network-allow flag and pass it through run-once.sh / wake-up.ts; (b) refactor run-once.sh so the Codex run produces output.md only, then a post-Codex shell step (outside the sandbox) reads output.md and runs `python -m ai_informer.notifications.cli`. Path (b) is simpler and decouples authoring from delivery — recommended for quick unblock.
2. Re-run PATH A validation (window-08) once the broadcast path works. Confirm `succeeded: feishu-default` in stdout and a real Feishu hit; then proceed to cron install.
3. Reconcile the `--cd` workspace vs `Runtime/runs/` canonical path issue (NEW work surfaced above). Either symlink-on-finalize or change the Codex `--cd` to workdir root.
4. Knowledge-promotion housekeeping pass (the 9-10 candidates above).
5. Historical webhook URL redaction sweep (≈53 leaks across 10 episode files).
6. Engine scaffold WT change commit decision (human-owned).
7. Adapter-health pickup queue: t4 (github_trending stars=0), then t6 (meta-ai-blog HTTP 400), then t7 (reddit-ai HTTP 403). Order is recommendation-only; the human can re-prioritize.

Monitored automatically:
- Nothing. Cron is not installed. The supervisor heartbeat (every 10 min, per Runtime/agent.json) keeps the agent alive and able to receive human messages, but no business pipeline fires automatically.

Needs human attention:
- Decision on broadcast unblock approach (path a vs path b above). Mailbox is await-reply.
- Whether PATH B (manual) delivery should resume on a daily cadence while PATH A is being fixed.
- Engine scaffold WT change commit decision (long-run-agent-harness/engine/scaffolds/basic-agent/).
- Adapter-health pickup order (after PATH A is unblocked).

References:
- Episode file: /home/ubuntu/agents/builders/ai-informer-builder/Memory/episodes/2026/05/ep--20260506T060000Z--build--ai-informer-path-a-validation-cron-install-and-completion-audit.md
- PATH A artifacts: /home/ubuntu/agents/builders/ai-informer-builder/.runs/2026-05-06T05-57-18-224Z-vjfu9a/Runtime/runs/08/20260506-0800/
- PATH B artifacts (verified working baseline): /home/ubuntu/agents/builders/ai-informer-builder/Runtime/runs/18/20260506-0453/
- Trajectory: /home/ubuntu/agents/builders/ai-informer-builder/trajectories/2026-05-06T05-57-18-224Z-vjfu9a.jsonl
- Build manual: /home/ubuntu/agents/builders/ai-informer-builder/implement_plan.md
- Design root: /home/ubuntu/agents/builders/ai-informer-builder/design/ai_informer_agent_design.md
