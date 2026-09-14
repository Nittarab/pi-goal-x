# Explicit execution contract (#55)

Supersedes the tool-name cooldown in PR #58. No tool, changed output, or unfinished goal implicitly authorizes another run. Each execution ends with an explicit ready/wait/complete/pause/blocked decision. Missing decisions receive at most one repair run, subject to allowance, then pause.

maxAutonomousRuns is a nonnegative safe integer in global/project settings; zero explicitly disables automatic runs, including when a positive global allowance is inherited. An absent project value inherits the global value; absent at both scopes disables automatic runs. Agents may configure settings. Persist consumed runs across messages, reloads, focus changes and settings edits; only goal creation or explicit user /goal-resume renews the period. The limit covers extension-generated kickoff, continuation, polling, event wake, repair and network recovery. It does not cap host tool loops or model turns started independently by another extension. Because agents may increase the ceiling, this is a configurable scheduling limit, not a hard spending cap. Existing token budgets still apply.

update_goal retains lifecycle forms and adds mutually exclusive continuation.ready(next_action) and continuation.wait(reason, deadline, optional polling interval_seconds/max_checks). Returned wait_id must be reused for subsequent checks; re-declaration cannot reset counters or deadline. Scheduling declarations persist before reporting success and terminate the execution segment. Additional model work invalidates a prior decision.

Persist ownership/generation, decisions, waits, counters and dispatch identity. Authorize atomically before every extension wake. Reject stale/duplicate messages; an ambiguous claimed dispatch after a crash requires resume. Follow real agent_start through agent_settled, including custom-message entry, queued messages, retries and compaction.

Waiting spends no model turns until a signal or explicit bounded check. Deadline/check/allowance exhaustion pauses without another model call. Restore owning-session waits without catch-up; require explicit resume for another session. Pause/complete/clear/focus/user takeover invalidate delivery. Waiting does not accrue active time. Closed sessions do not execute timers.

An outstanding wait deadline applies to every extension dispatch, including network recovery and missing-disposition repair after a polling check. Recheck the deadline immediately before claiming a dispatch, even if readiness or recovery was scheduled before expiry. User steering: do not change README.md in this review follow-up; preserve the README already present on PR #58 before the review fixes.

Document pi-goal:wake {goalId, waitToken}: matching signals authorize a wake; stale/duplicate signals do nothing. Register before starting a producer, or retain its result until registration. Ordinary third-party follow-ups remain host work and invalidate pending scheduling; their model spending is outside this gate.

Show next action/wait reason, timing/check counts, and allowance usage in dashboard, status and get_goal. /goal-resume means continue now and renew, but cannot invent a limit. Revise PR #58; do not merge, bump version or publish.

Context constraint (user steering): automatic scheduling remains off by default and agents may enable it through settings. Keep the disabled-mode prompt short; include detailed scheduling guidance only when an allowance is configured. Measure and publish overhead; avoid duplicate schema examples in the prompt.
