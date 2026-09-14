# Milestones — lifecycle safety kernel

## Baseline (step 1, 2026-09-14)

Workdir `/home/ubuntu/git/work/pi-goal-x`.

- Branch `improve/lifecycle-safety-kernel` at `323fd4c`
  (`Merge pull request #62 from tmonk/codex/release-0-31-4`, package 0.31.4).
- Working tree clean.
- `origin` `https://github.com/Nittarab/pi-goal-x.git`; `origin/main` = `323fd4c`.
- `upstream` `https://github.com/tmonk/pi-goal-x.git`.
- Local `main` was stale (`6c72fc8`); not used.

Canonical checks on that commit:

- `python3 -B scripts/test-update-ranking.py` — pass
- `npm run check` — pass
- `npm run lint` — pass
- `npm run test:all` — 993 pass, 0 fail, 0 skip
- `npm run test:selfcheck` — 944 pass
- `npm pack --dry-run` — `pi-goal-x@0.31.4`, 66 files
- `npm audit --omit=dev` — 0 vulnerabilities
- `npm run bench:gate:naf` — PASS

Read AGENTS.md, `specs/2026-09-14-hygiene-continuation/`,
`/tmp/pi-goal-session-forensics.md`. Did not duplicate scheduler/empty-turn work.

## v0.31.4 concern re-test (before code)

- Default `networkRecovery.maxAttempts` is `0` (unbounded). Exhaustion path
  notifies and leaves the goal active (`goal-events.ts` agent_settled).
- Auditor `createAgentSession` tools include `bash`. Oracle does not.
- `pauseActiveGoal` does not charge; agent pause does.
- `/goal-refresh` invalidates caches + `updateUI` only.

## Spec (step 2)

Wrote `PRODUCT.md` and `TECH.md` defining PR A/B/C, issues, and acceptance
before implementation.

## PR A implementation

- Default `networkRecovery.maxAttempts` is 5; 0 remains unbounded.
- Exhaustion pauses with pauseReason and `/goal-resume` hint.
- Auditor tools: `read`/`grep`/`find`/`ls` only.
- `pauseActiveGoal` charges elapsed seconds first.
- `/goal-refresh` drops the persisted pool snapshot, reconciles the focused
  goal, and reinstalls the tool profile when `disableTasks` changed.

Validation on this branch: check, lint, test:all 997 pass (was 993), selfcheck
947, pack dry-run, audit 0, bench:gate:naf PASS, ranking tests OK. No version
bump, no merge, no publish.

PR: https://github.com/Nittarab/pi-goal-x/pull/1

## PR B implementation

- In-turn `apply`/`updateTask` flush or take the lock path before success.
- Held lock returns `ok: false` instead of a successful buffer.
- `refreshFocusedFromAuthoritativeFile` parses the goal file for checkpoint
  schedule/claim, ignoring the pool snapshot.
- Tests: held lock, missing endTurn, stale memory vs disk pause, competing writer.

Validation: `npm run test:all` 998 pass.

PR: https://github.com/Nittarab/pi-goal-x/pull/2

## PR C implementation

- Prompt/tool guidance: parent-child is decomposition; alternative paths are peers.
- Paused goals with tasks advertise `update_goal_task`; skip/complete work while paused.
- `set_goal_tasks` cannot drop pending tasks when `blockCompletion` is on.
- `/goal-status health` includes scheduler phase/dispatch and why work is not queued.

Validation: `npm run test:all` 1002 pass.

PR: https://github.com/Nittarab/pi-goal-x/pull/3

## Issues (step 6)

Enabled Issues on `Nittarab/pi-goal-x` (they were off). Filed:

1. https://github.com/Nittarab/pi-goal-x/issues/4 — Concurrent goal execution needs cross-session ownership leases.
2. https://github.com/Nittarab/pi-goal-x/issues/5 — Do not mix pi-subagents mission status with pi-goal-x goals (goal `mtyaft58-00lozy` vs mission `dbd35a5a-a12b-4523-8ce4-39f6c818c6f4`).

## Remaining risks

- Default recovery cap of 5 may surprise unbounded-retry users (`networkRecovery.maxAttempts: 0`).
- Auditor cannot run `bash` verification.
- Immediate durable writes increase lock traffic vs the old in-turn buffer.
- Gate flattening uses wording heuristics (gate/fibonacci/prove-one/full-matrix); true decomposition without those words still nests.
- Cross-session leases are not implemented (issue #4).

## Manual QA

1. Force provider 503s: expect `recovery 1/5` then paused + `/goal-resume`.
2. Pause after a few seconds; `activeSeconds` increases.
3. External-edit a goal file and `disableTasks`; `/goal-refresh` applies both.
4. Pause a `blockCompletion` goal; skip/complete with `update_goal_task`; dropping a pending id via `set_goal_tasks` refuses.
5. Nested `prove-one` / `fix-missing` / `full-matrix`: complete `prove-one` without completing `full-matrix`; skip the unused path.
6. `/goal-status health` shows scheduler and a dispatch reason when idle/waiting. Widget must not say running while waiting.
