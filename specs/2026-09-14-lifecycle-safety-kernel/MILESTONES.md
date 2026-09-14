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
