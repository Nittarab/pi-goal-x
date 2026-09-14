# Continuation cooldown (#55)

Prevent read/search/bookkeeping-only runs from injecting full-context checkpoints at model round-trip speed. Default continuationIdleDelayMs is 300000 (five minutes), with project/global settings and PI_GOAL_CONTINUATION_IDLE_DELAY_MS override; zero explicitly restores immediate follow-ups. The limit is 2147483647 ms to avoid timer overflow.

Reads and searches remain useful work. Their tool chains run normally; only the next autonomous run after settlement waits. Runs containing write/edit/bash retain immediate cadence. Shell commands are opaque: this policy does not claim to detect no-op shell commands or semantic progress. The no-tool gate from #54 stays intact.

Creation, resume, session kickoff, compaction and network recovery keep their existing timing. User input or a producer-delivered follow-up cancels the sleeping checkpoint and runs normally. Pausing, stopping, changing focus, disabling auto-continue or closing the session prevents stale delivery. The cooldown is an in-session timer, not a durable scheduler.

Use the existing pending-message and before-agent-start lifecycle for background completions; do not subscribe to undocumented third-party producer events or infer that all work must wait while any child is running. The optional producer-specific gating in the issue is not necessary to prevent hygiene polling loops.

Show a concise notification when a run enters cooldown so users understand why an active goal is waiting. Open a PR; do not merge or release it in this task.
