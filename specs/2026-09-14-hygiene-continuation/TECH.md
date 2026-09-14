# Design

Track write/edit/bash attempts alongside the existing meaningful-work flag for the whole agent run. Clear both before_agent_start. Remove normal turn_end scheduling; only agent_settled schedules successful run continuations, after agent_end reconciles goal ownership and error/abort state.

Pass an explicit delay through GoalCore to GoalRuntime. force controls dedup only, not the supplied delay. The timer waits once, then polls host readiness at the existing 50 ms cadence. Existing cancellation paths clear it; new input and producer follow-ups use those same paths. Clamp runtime delay defensively to the timer range; reject invalid configuration including overflow and preserve settings provenance/cache invalidation.

Validate exact timer deadlines with fake timers, real event ordering, repeated hygiene runs, earlier writes plus text-only final turns, reset after work, zero override, errors/recovery, user and background wake, cancellation and goal changes. Exercise setting parsing, layering, save/load, environment and report/UI exposure; run full CI checks.
