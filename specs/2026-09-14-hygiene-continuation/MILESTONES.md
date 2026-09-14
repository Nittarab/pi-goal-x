Reviewed #55 and the referenced fork. Selected a nonzero default to protect existing users, run-scoped classification to preserve multi-turn work, and existing message lifecycle for producer wakeups.

Implemented run-scoped cooldown and removed the earlier turn_end scheduling path. Added project/global/environment settings, menu/report support, strict bounds and cache key participation, user notifications, README and changelog.

Regression work exposed permissive legacy string parsing (new setting now validates full decimal input), an obsolete pre-settlement continuation assertion, settings menu row-count changes, and test expectations for cache refresh/provenance. Updated the tests to follow existing settings cache invalidation and source naming.

Real SDK worker now exercises repeated read-only runs, delayed automatic continuation, immediate user input and triggerTurn/followUp producer delivery without duplicate checkpoints. The worker passes with eight provider requests and no model credentials. Fake timers cover 300000 ms boundaries, deduplication, cancellation, readiness polling, explicit kickoff, no-tool stopping, run reset and the zero override. Context gate, six SDK payload checks, NAF gate, production audit and package dry run pass.

Final local validation: 974 tests pass with no skips; test-manifest self-check and 927 unit tests pass. TypeScript, ESLint and git diff whitespace checks pass. No dependency or package version change. Prepared for PR review, not merge/publication.
