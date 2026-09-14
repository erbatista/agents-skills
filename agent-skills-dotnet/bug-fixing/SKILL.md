---
name: bug-fixing
description: Diagnose and fix existing defects in any codebase by reproducing the problem, establishing root cause with evidence, adding regression coverage when practical, and applying the smallest safe correction. Use for crashes, exceptions, incorrect results, regressions, intermittent failures, state and concurrency bugs, UI that does not update, and production incidents, even when the user only says "this doesn't work". Language-agnostic; detects the stack and loads the matching stack skill (dotnet-wpf, blazor) for framework-specific investigation and validation. Do not use as the primary workflow for net-new feature development; use feature-development for that.
---

# Bug Fixing

Restore correct behavior with the smallest safe change, prove the root cause, and keep the defect from returning.

Precedence when instructions conflict, highest first:

1. explicit user instructions
2. the repository's agent file (`AGENTS.md`, `CLAUDE.md`, or equivalent)
3. the stack skill loaded in step 0
4. this skill

## Core rule

Do not jump from symptom to patch. The required chain is:

`Observed behavior → Reproduction → Evidence → Root cause → Regression coverage → Fix → Verification`

A plausible story is not enough. Establish the root cause with code-path evidence, a failing test, a minimal reproduction, or another observable signal before changing production code.

## 0. Identify the stack

Do this before reading any code.

1. If the repository's agent file names a stack skill, load it. That declaration wins over detection.
2. Otherwise inspect the repository root:
   - `*.sln` or `*.csproj` containing `<UseWPF>true</UseWPF>` → load `dotnet-wpf`
   - `*.csproj` alongside `.razor` files → load `blazor`
   - any other stack with no matching skill → continue with this workflow alone
3. Follow the stack skill's "What to read, when" section. It says which reference to open while investigating and before validating.
4. Note the outcome for the report: `Stack skill: dotnet-wpf`, `blazor`, or `none`.

When both WPF and Blazor projects exist in one solution, load the skill for the project where the defect is observed; switch if the investigation crosses into the other.

## 1. Characterize the defect

Extract expected behavior, actual behavior, the trigger sequence, affected scope, any error messages, stack traces, logs, or screenshots supplied, and the first known bad version or recent related changes.

Translate the report into a concrete technical statement: crash vs incorrect result; deterministic vs intermittent; presentation-only vs application-state defect; first occurrence vs regression.

## 2. Reproduce or build a minimal proof

Search for an existing test or reproduction path first. Then reproduce using the narrowest relevant path, capture the observable failure, identify the state transition and call path, and reduce unrelated variables.

If the defect cannot be reproduced, continue with static investigation and the available evidence, and mark the reproduction gap explicitly. Do not invent a successful reproduction.

## 3. Investigate systematically

Inspect only the code needed to test plausible hypotheses. Stack-independent areas: recent changes and their callers, state transitions, nullability and lifetime assumptions, exception paths, async flow and cancellation, shared mutable state, caching, persistence and serialization, dependency-injection lifetimes. The stack skill's bug checklist lists the framework-specific ones.

Ask whether the visible symptom is downstream of a state, binding, or threading problem rather than in the component where it appears.

## 4. State the root cause before fixing

Use this shape:

**Cause:** what is wrong
**Mechanism:** why it produces the observed behavior
**Evidence:** what in the code, test, or log supports the conclusion

Example of an acceptable statement:

> **Cause:** `OrderListViewModel.LoadAsync` assigns the result of every in-flight load to `Orders`, with no cancellation or sequence guard.
> **Mechanism:** Changing the filter twice quickly starts loads A then B; A completes after B and overwrites the list with rows for the old filter.
> **Evidence:** No token or request counter in `LoadAsync`; a test that completes two awaited loads out of order leaves the first filter's rows in `Orders`.

When confidence is low, say so and list the competing hypotheses. Do not present a guess as fact.

Stop and ask before changing code when:

- the root cause is in a shared component and the fix would change behavior for other callers
- the fix changes a public API, persistence format, or threading model
- you could not reproduce and the fix would be a guess

If the environment cannot ask questions (non-interactive run), end the turn with the root-cause statement and the options instead of a speculative patch.

## 5. Add regression protection

Encode the defect in a test before applying the fix when the framework and failure mode make that practical. The test should fail against the pre-fix code, pass after the fix, and target the defect's contract rather than incidental implementation. Choose the lowest-level reliable boundary; a service or ViewModel test is preferable to a UI test when it fully proves the failure.

If a test is impractical, say why and use the strongest available validation. If the repository has no test project, do not create one unless asked; report the gap.

## 6. Implement the smallest safe fix

Change only what corrects the root cause. Preserve unrelated behavior, follow existing architecture and patterns, and avoid opportunistic refactors unless needed to make the fix correct, testable, or safe.

Do not mask symptoms: no broad exception handling, arbitrary retries, sleeps, thread hops, or null suppression whose only purpose is to make the symptom disappear. For concurrency problems, fix ownership, ordering, or cancellation rather than hiding the race. Framework-specific fix guidance is in the stack skill's bug checklist.

## 7. Verify the defect and the fix

Run, in order: the new or updated regression test, relevant unit and integration tests, the affected build, broader tests when practical, and targeted runtime validation for behavior only observable at runtime. Use the stack skill's validation reference for the commands and environment rules.

For intermittent or timing-sensitive defects, repeat or stress the test when the repository provides a mechanism.

Do not claim the defect is fixed unless the strongest available evidence supports it. Never report a check as passed if it did not execute.

## 8. Review for unintended consequences

Inspect the final diff and answer: Does the change address the stated root cause? Could it alter adjacent code paths? Did it introduce a threading, lifetime, or cancellation problem? Does the regression test prove the important boundary? Did unrelated files or formatting change? Could the bug reappear through an equivalent path?

## 9. Report

Use this template exactly:

```
## Root cause
**Cause:** <...>
**Mechanism:** <...>
**Evidence:** <...>

## Fix
<what changed and why it is the smallest safe correction>

## Regression coverage
<test added/updated and the boundary it proves> | none (why)

## Stack skill
<dotnet-wpf | blazor | none>

## Validation
- ran: <commands, each with pass/fail>
- not run: <commands and why> | none

## Remaining uncertainty
<reproduction gaps, competing hypotheses, follow-up work> | none
```

Keep it evidence-based; distinguish observed facts from assumptions.

## Guardrails

- Do not rewrite the component unless the defect cannot be safely corrected within the existing design.
- Do not close the issue because the symptom disappeared without understanding why.
- Do not add logging as a substitute for analysis when existing evidence is sufficient.
- Do not change public APIs, persistence formats, or threading models without considering compatibility and callers.
- When evidence points to a deeper architectural problem, fix the defect safely first and identify the larger remediation as separate work.
- If the request mixes a bug and an enhancement, fix the bug here first, then switch to `feature-development`; keep the two diffs separable.
