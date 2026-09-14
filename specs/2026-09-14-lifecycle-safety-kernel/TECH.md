# Technical — lifecycle safety kernel

Date: 2026-09-14

Stack three independently reviewable PRs on `improve/lifecycle-safety-kernel`
(base `323fd4c` / v0.31.4). Target `Nittarab/pi-goal-x` `main`. Do not merge.

## PR A — runtime safety

### Finite recovery

- Change `DEFAULT_NETWORK_ERROR_RECOVERY_POLICY.maxAttempts` and the layered
  settings default for `networkRecovery.maxAttempts` from `0` to `5`.
- Keep `0` as unbounded (existing `networkErrorBackoffPlan` semantics).
- Export a named default constant; settings menu/report labels must say that
  `0` is unbounded and the default is `5`.
- On `agent_settled`, when `scheduleNetworkErrorRetry` returns no plan,
  persist pause (`status: paused`, `autoContinue: false`) with
  `pauseReason` describing recovery exhaustion and
  `pauseSuggestedAction` pointing at `/goal-resume`. Charge elapsed
  accounting first. Notify; do not leave the goal `active`.
- Use existing `stopReason: "agent"` rather than adding a new status.
- Update unit/lifecycle tests that currently assert unbounded default and
  “goal remains active” on exhaustion. Do not change the delay ladder.

### Read-only auditor

- In `runGoalCompletionAuditor`, register `tools: ["read", "grep", "find", "ls"]`
  only. Drop `bash`. Update `buildGoalAuditorPrompt` / labels that mention bash.
- Fault test: the session factory receives no `bash`/`write`/`edit`; a
  workspace file created before the audit is unchanged after a simulated
  auditor that would have used bash if present.
- Do not weaken Oracle isolation.

### Accounting before pause

- `pauseActiveGoal` must call `accountProgress` before `stopActiveGoal`.
- Recovery-exhaustion pause uses the same charge-then-clear order.
- Agent pause already charges; keep that. Add a regression that user pause
  after N elapsed seconds persists `usage.activeSeconds += N`.

### Immediate `/goal-refresh`

- After invalidating pool/ledger/settings caches, call
  `reconcileFocusedGoalFromDisk`, reinstall the tool profile when
  `disableTasks` changed (same hook as the settings menu), then `updateUI`.
- Integration test: external goal-file and settings edits become the live
  focused record and effective `disableTasks` profile without restart.

### Changelog and checks

- Document under `[Unreleased]`. No version bump.
- Canonical checks: ranking tests, `npm run check`, `lint`, `test:all`,
  `test:selfcheck`, `npm pack --dry-run`, `npm audit --omit=dev`,
  `bench:gate:naf`.

## PR B — durability (branch stacked on A)

Re-test v0.31.4 first: checkpoint authorization vs pool cache, mutation
success vs file commit, crash of accepted mutations.

Prefer a small durable commit at the existing `GoalService.apply` boundary.
Add two-session, held-lock, and crash-boundary tests. Introduce a journal
only if those tests prove an immediate commit cannot preserve accepted
mutations.

## PR C — task and operator correctness (stacked on B)

Re-test v0.31.4 first: drafting/task guidance, Fibonacci-as-parent, paused
tool profile vs `blockCompletion`, `set_goal_tasks` bypass, evidence merge,
`/goal-status health`.

Change prompts/guidance and tool-profile rules; preserve task status/evidence
on approved plan changes. Health output must include scheduler phase, last
dispatch/outcome, retry/recovery state, and why work is not queued.

Session JSONL evidence: entries 1390–1395 and 1750–1755 of
`/tmp/pi-goal-session-01a08071.jsonl`.

## Issues

Inspect current claim/ownership before filing. If two sessions can both
dispatch on one goal, file an ownership/lease issue. File a separate
pi-subagents mission-integration issue; do not implement it here.
