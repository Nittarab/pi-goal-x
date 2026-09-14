# Execution contract context cost

The committed baseline is intentionally updated for the explicit execution contract. CONTEXT-BEFORE.json preserves the previous main baseline; CONTEXT-AFTER.json is the new measurement from the same 24 fixtures. The gate still remeasures and checks exact breakdowns, semantic counts, single objective blocks, and filtered historical checkpoints.

| Measurement | Before | After | Increase |
| --- | ---: | ---: | ---: |
| Serialized request characters (24 fixtures) | 242,142 | 258,720 | 16,578 (6.85%) |
| Extension-attributable characters | 130,042 | 146,620 | 16,578 (12.75%) |
| Estimated tokens (characters / 4, rounded per fixture) | 60,543 | 64,689 | 4,146 |
| Child request characters | 15,225 | 15,225 | 0 |
| Active regular, 10 tasks | 12,238 | 13,201 | 963 |

The typical active-request increase comprises 714 tool-schema characters, 224 execution-policy/state characters, and 25 SDK tool-guidance characters. This is about 241 estimated tokens per such request in default disabled mode. Detailed scheduling policy is conditional on a configured allowance and adds fewer than 250 characters. The schema remains available so agents can enable the feature via settings without reload. These are deterministic serialization measurements, not billed-token measurements. The schema and disabled-mode instructions were shortened following user feedback; the first implementation added 1,833 characters to this active fixture.

The extra schema and instructions make the scheduling contract available to the model; removing them would leave the new tool forms and allowance semantics undiscoverable. A scheduling declaration is itself an additional model-generated tool call and persisted tool result at a segment boundary; its dynamic output and task-dependent frequency are not represented by this static baseline. This design does not claim lower total token use per completed goal.

The separate real SDK worker measures current-state refreshes on custom-message runs (which bypass before_agent_start): 176 serialized content characters for ready state, 322 for a repair, and 301 for the fixture wait. It reuses the inherited objective/policy block when present. Its eight request sizes were 12,100; 12,370; 13,090; 11,783; 14,159; 12,852; 14,630; and 14,834 characters. Those include dynamic tool history and are not a controlled before/after savings comparison. Four extension dispatches are persisted as four used allowance units; internal tool turns are not additional units.

Run allowance bounds extension-initiated executions only. It neither proves progress nor caps a third-party extension's direct host work or an indefinitely running host tool loop.

PR review follow-up: clarify the disabled-mode instruction from “is set” to “> 0” now that zero explicitly disables inherited automation. This removes three serialized characters from each of 14 active fixtures (42 total) without changing schemas or semantic counts. Remeasured the baseline and CONTEXT-AFTER.json; the gate retains its exact comparisons. The SDK request sizes above are historical measurements from the initial implementation.
