# Scheduler implementation

Use a dedicated persisted scheduler with the GoalService atomic mutation boundary; finish buffered turns before authorizing declarations or dispatch. All extension-generated messages pass a single authorize/claim/send gate. Persist generation-tagged claim before delivery, consume it before execution, and fail closed after ambiguous recovery. Runtime timers supply readiness polling/network delay only; they must not authorize work.

Track actual agent_start through final settlement (not before_agent_start). Preserve a logical execution across internal retries, invalidate a declared disposition if another model turn occurs, and keep checkpoint admission separate from normal user/producer takeover. A matching wait signal can be recorded before settlement, but delivery occurs only afterward. Wait checks retain the same wait identity and finite allowance. Resume explicitly resets epoch and consumption.

Keep lifecycle GoalStatus separate from a versioned scheduler field. Strictly validate persisted scheduler data; corrupt scheduling state requires resume rather than resetting counters. Read legacy records without scheduling them. Render bounded scheduler summaries through existing surfaces. Add maxAutonomousRuns to layered settings and remove the unreleased cooldown entirely.

Validation: pure scheduler fake-clock tests plus real GoalService and real SDK cases for tool independence, claim/cancellation races, missing decisions, owned restoration, external events, dynamic allowance, retries and compaction. Run full checks and measure extra model-context cost, updating baseline only for reviewed intentional contract overhead.

Use a compact disabled-mode policy and conditional enabled guidance, with allowance presence in the prompt cache key. Keep the scheduling schema discoverable so agents can enable the setting without reload; avoid dynamic tool-registration churn. Deduplicate tool descriptions and system instructions, retain validation at the mutation boundary, and remeasure context.
