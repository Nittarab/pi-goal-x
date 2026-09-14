## Superseded cooldown implementation (historical)

Reviewed #55 and the referenced fork. Selected a nonzero default to protect existing users, run-scoped classification to preserve multi-turn work, and existing message lifecycle for producer wakeups.

Implemented run-scoped cooldown and removed the earlier turn_end scheduling path. Added project/global/environment settings, menu/report support, strict bounds and cache key participation, user notifications, README and changelog.

Regression work exposed permissive legacy string parsing (new setting now validates full decimal input), an obsolete pre-settlement continuation assertion, settings menu row-count changes, and test expectations for cache refresh/provenance. Updated the tests to follow existing settings cache invalidation and source naming.

Real SDK worker now exercises repeated read-only runs, delayed automatic continuation, immediate user input and triggerTurn/followUp producer delivery without duplicate checkpoints. The worker passes with eight provider requests and no model credentials. Fake timers cover 300000 ms boundaries, deduplication, cancellation, readiness polling, explicit kickoff, no-tool stopping, run reset and the zero override. Context gate, six SDK payload checks, NAF gate, production audit and package dry run pass.

Final local validation: 974 tests pass with no skips; test-manifest self-check and 927 unit tests pass. TypeScript, ESLint and git diff whitespace checks pass. No dependency or package version change. Prepared for PR review, not merge/publication.

## Wholesale revision
User rejected tool-based classification and approved explicit ready/wait outcomes, bounded checks, and configurable allowance with no default automatic execution. Agents may configure the ceiling; edits never reset consumption. Replaced PRODUCT and TECH before implementation.


Implemented a versioned scheduler separate from goal lifecycle, with durable GoalService claims, session ownership and generation checks, configurable consumed-run accounting, explicit ready/wait declarations, one-shot repair, and finite polling/event waits. Removed tool classification from scheduling and removed the cooldown setting. Creation and explicit resume are the only allowance-period boundaries. Terminal lifecycle operations and incoming user/producer work invalidate scheduling intent.

SDK validation exposed that custom-message entry skips before_agent_start. Execution tracking now uses agent_start through agent_settled; custom runs receive current scheduling state without repeating an inherited objective block. The actual SDK fixture makes eight provider requests over an initial user execution and four extension dispatches, including mixed write/read/tool work, a waiting producer wake, and one unsuccessful repair. Replaying a historical dispatch produces no ninth request. Waiting adds no requests. Existing real SDK producer and network-recovery suites remain covered.

Race testing found that an old owner's readiness callback could otherwise claim a newer owner's decision. Readiness now carries its original generation and atomically rejects changed ownership/generation without pausing the new owner's goal. Persistence failure never returns terminate/success. Claims survive failed delivery as spent allowance and are not replayed after reload. Fake-clock tests cover early/duplicate wakes, deadlines, counters, settings edits, ownership transfer, subsequent model work, idle readiness, and active-time suspension.

Intentional context drift initially failed context:gate. Added before/after evidence and CONTEXT.md rationale, then updated the deterministic baseline without weakening any invariant. The new decision schema and instructions cost context; no progress or token-saving claim is made for them. Static historical benchmark gates are regression checks, not new end-to-end performance measurements.


Final lifecycle review preserved the admitted action in running-state context so contract-repair instructions reach the actual provider. The consumed claim remains unusable for another dispatch. Explicit resume during host work now waits for settlement and retains kickoff intent unless superseded by subsequent model work. Added regressions for both.

Added an actual SDK retry/compaction variant: ten provider requests (eight work requests, one transient failure, one summary), one native retry, four consumed extension runs. The first compaction attempt correctly rejected the fixture as too short; adding a persisted turn boundary makes it compactable. Compaction preserves the live wait and does not consume allowance.

User requested low context overhead and off-by-default operation. Updated PRODUCT then TECH, shortened duplicate policy/schema descriptions, and made detailed guidance conditional on configured allowance. Default mode remains discoverable by agents through a short settings instruction. Typical default overhead fell from 1,833 to 966 characters (about 242 estimated tokens); no tool-schema hiding or reload is needed to enable it. A cache regression verifies settings changes select the right policy.

Final validation: 982 tests pass, zero skips/failures; manifest self-check and 934 unit tests pass. TypeScript, ESLint, context gate (24 fixtures), provider cross-check (six real SDK payloads), NAF/runtime-token/comprehensive benchmark gates, ranking tests, production audit (zero vulnerabilities), package dry run and whitespace checks pass. No package version or dependency changes. PR remains open; no merge or publication.
