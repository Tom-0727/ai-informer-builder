---
id: kn.scheduler-deployment.heuristic.foreground-validate-before-install
kind: heuristic
summary: Before installing a scheduled trigger (cron / systemd timer / etc.) for an autonomous flow with new code paths, run at least one foreground end-to-end pass first — the cost of one foreground run is much smaller than recovering from a silent first cron failure.
last_edited_at: 2026-05-07
status: active
tags: [scheduler, cron, validation, deployment, debugging]
---

## Core Content

- Discipline (call it OPTION X): when a flow is about to be wired into a scheduled trigger and contains code paths that have not been exercised end-to-end since the last validation, the next action is a foreground run of the same entrypoint that the trigger will invoke, NOT the trigger install. Only install the trigger after the foreground run passes its observable success criteria.
- The scheduled-trigger trap: a silent first failure inside cron at e.g. 08:00 surfaces only as "no output, no notification, no artifact" — the trajectory is not captured anywhere with high fidelity. Reconstructing what happened means correlating sparse cron mail, system logs, and partial workspace artifacts. A foreground run with the same entrypoint executes in roughly the same wall-clock time but with full stdout/stderr, exit code, and on-disk intermediate state observable in real time.
- The asymmetry: a 9-minute foreground validation that surfaces a sandbox DNS misconfiguration is dramatically cheaper than a day's gap between a 08:00 cron silent failure and the human noticing no morning briefing. The foreground cost is bounded and known; the post-failure debugging cost is unbounded and unobservable until it has already accrued.
- Generalizes to any autonomous flow with new code paths since last validation: cron jobs, systemd timers, GitHub Actions schedules, k8s CronJobs, cloud scheduler invocations.

## Boundary Conditions

- Applies when the scheduler's runtime observability is sparse (cron mail, plain log file, no live trajectory capture) AND the flow has new code paths since the last successful end-to-end run.
- Less relevant when the flow has already been foreground-validated within the same change cycle and no new code path has been added since.
- Less relevant when the scheduled trigger has rich live observability — a managed scheduler with structured logs, retries, alerting, and trajectory capture compresses the asymmetry.

## Limitations

- Foreground validation only catches deterministic failure modes. Race conditions, time-of-day-dependent failures (DNS at 04:00 vs 14:00, upstream rate-limit windows), and resource-contention failures may still slip past a single foreground run.
- The foreground validation cost is non-trivial when the flow is long-running; for multi-hour flows the heuristic shifts toward "validate the new code path in isolation" rather than "validate the full end-to-end".
- The discipline is procedural and drifts. Without a checklist or a structural gate that blocks the trigger install before validation, the rule degrades over time.

## Optional Notes

- Provenance: ep.20260506T060000Z is the canonical instance — PATH A foreground validation surfaced a sandbox DNS issue in 9 minutes that would otherwise have manifested as a silent first failure inside the daily cron.
- Cross-reference: kn.codex-sdk.heuristic.sandbox-network-requires-explicit-mode — the operational consequence that PATH A specifically caught.
