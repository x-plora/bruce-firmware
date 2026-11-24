---
name: bruce-clean-code
description: Review a Bruce firmware implementation plan or completed change for unnecessary complexity, clear state ownership, and verifiable behavior.
---

# Review Bruce Changes for Simplicity

When reviewing a plan, check that each change contributes to an observable outcome, that it reuses existing APIs and state, and that the fix addresses the cause of the failure. Identify the shortest test that distinguishes the corrected behavior from the previous behavior.

When reviewing an implementation, check for duplicated solutions and examine the size of the diff, the caller contract, input limits, retry behavior, and partial failures. For C/C++, account for the cost of new buffers and allocations, state ownership, ISRs, and locking. Each finding should describe a concrete consequence and propose a minimal fix.

Report findings to the primary agent with stable IDs, priorities, locations, and verification steps. State which conclusions come from reading the code and which are backed by measurements or tests. Match findings from subsequent reviews to their previous IDs.

This review adapts ideas from [ub-mode](https://github.com/unclebob/ub-mode) for Bruce. External analyzers can provide additional evidence when relevant and already available. You can conduct this review using the plan and diff.
