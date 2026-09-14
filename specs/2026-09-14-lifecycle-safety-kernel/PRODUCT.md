# Lifecycle safety kernel

Date: 2026-09-14

Safety-first follow-up to v0.31.4 (`323fd4c`). Upstream already merged the
empty-turn auto-continue fix and the explicit persisted scheduler. This work
does not duplicate or revert that contract. Each PR is independently safe and
reviewable; none is merged or published from this effort.

Incident context: session `01a08071` / goal `mtyaft58-00lozy` (forensics in
`/tmp/pi-goal-session-forensics.md`). Mission `dbd35a5a-a12b-4523-8ce4-39f6c818c6f4`
is a pi-subagents concern and must not be attributed to pi-goal-x.

## Invariants (all PRs)

- At most one authorized goal dispatch exists.
- Paused, completed, cleared, stale, expired, or foreign-session goals cannot dispatch work.
- Automatic cost is bounded after provider failure.
- A successful state-changing tool result survives restart.
- The auditor cannot modify tracked or untracked workspace files.
- UI status must not say running when the scheduler is not able to run.
- Do not add features unrelated to these failures.

## v0.31.4 re-test (current main, before code)

Confirmed against this checkout:

1. **Provider recovery is unbounded by default.** `networkRecovery.maxAttempts`
   resolves to `0` (unbounded). After a configured cap, the goal stays
   `active` with a warning; it is not paused. Automatic cost is therefore
   unbounded unless the user opts into a cap.
2. **Auditor bash is unrestricted.** Isolated auditor sessions register
   `tools: ["read", "grep", "find", "ls", "bash"]`. Read-only is prompt text
   only. Oracle already omits bash/write/edit.
3. **User pause drops elapsed seconds.** `update_goal({status:"paused"})`
   charges `accountProgress` first. `pauseActiveGoal` (Esc, `/goal-pause`,
   user abort) clears accounting without charging the open interval.
4. **`/goal-refresh` invalidates caches and notifies, then `updateUI`.** It
   does not reconcile the focused goal from disk, reinstall the tool profile,
   or otherwise apply refreshed settings to the live session.

## PR A — runtime safety (`improve/lifecycle-safety-kernel`)

### Finite provider/network recovery

- Default recovery is finite: five attempts along the existing 5/10/20/40/80s
  ladder (plateau still `maxDelayMs`, default 80s).
- `networkRecovery.maxAttempts` `0` remains an explicit unbounded opt-in.
- After the default or configured cap is exhausted, stop retrying. Persist a
  paused, resumable goal (`status: paused`) with a visible pause reason and
  suggested `/goal-resume` when the provider is healthy. Do not leave the goal
  `active`. Do not claim the UI is running.
- Successful turns still reset the in-memory recovery counter. User abort,
  pause, clear, focus change, and shutdown still cancel pending recovery.

### Read-only completion auditor

- Isolated auditor sessions must not receive `bash`, `write`, or `edit`.
- Tool list is `read`, `grep`, `find`, `ls` only (same isolation idea as the
  Oracle). Prompt text is not a boundary.
- The auditor cannot create, modify, or delete tracked or untracked workspace
  files.

### Charge elapsed accounting before pause

- Every path that pauses an active goal (user abort, Esc, `/goal-pause`,
  agent `update_goal({status:"paused"})`, recovery exhaustion) charges the
  open elapsed interval before clearing the accounting baseline.
- Charging remains idempotent; a second pause does not double-count.

### `/goal-refresh` applies immediately

- After cache invalidation, the live session must use refreshed pool, focused
  goal, ledger, and settings: reconcile focused goal from disk, reinstall the
  tool profile when `disableTasks` changed, refresh UI, and use the new
  settings on the next scheduling/recovery decision without a restart.
- Still no watchers or per-turn polling.

## PR B — durability (stacked on A)

- Authorize each automatic checkpoint from authoritative current disk state,
  not a stale cross-process pool cache.
- Do not report a lifecycle or task mutation as successful until the
  authoritative file commit succeeds.
- Preserve or recover accepted state mutations across a process crash.
- Prefer a small immediate durable commit for critical mutations over a large
  journal unless evidence proves a journal is necessary.

## PR C — task and operator correctness (stacked on B)

- Parent-child means decomposition, not ordering or conditional branching.
- A Fibonacci-style gate becomes peer tasks; the non-selected path can be skipped.
- A paused goal with `blockCompletion` must not dead-end: the safe per-task
  closeout tool remains available, or completion is not blocked on tasks while
  that tool is hidden.
- Task-list replacement must not exist only to bypass completion.
- Approved plan changes preserve task status and evidence.
- `/goal-status health` includes scheduler state, last dispatch/outcome, retry
  state, and a clear reason when work is not queued.
- Regressions from session entries 1390–1395 and 1750–1755.

## Issues (not code in these PRs)

- One issue: concurrent execution ownership/leases if v0.31.4 does not already
  prevent two sessions from doing work on the same goal.
- One separate issue: pi-subagents mission integration (goal
  `mtyaft58-00lozy` vs mission `dbd35a5a-…`). Do not misattribute mission
  notices to pi-goal-x.

## Non-goals

- Do not merge PRs or publish a package.
- Do not change README.md unless a later product decision requires it.
- Do not revert or reimplement the explicit scheduler or empty-turn fix.
- Do not add unrelated features, journals-by-default, or mission-ledger work.
