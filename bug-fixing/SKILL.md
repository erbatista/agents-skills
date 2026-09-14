---
name: bug-fixing
description: Diagnose and fix existing C#/.NET/WPF defects by reproducing the problem, establishing root cause, protecting the defect with regression coverage when practical, and applying the smallest safe correction. Use for crashes, incorrect behavior, binding failures, threading issues, state bugs, regressions, exceptions, and production defects; do not use as the primary workflow for net-new feature development.
---

# Bug Fixing

## Purpose

Restore correct behavior with the smallest safe change while proving the root cause and preventing the defect from returning.

Treat repository conventions, `AGENTS.md`, existing tests, logs, issue details, and established architecture as the source of truth. User instructions take precedence over this skill when they conflict.

## Core rule

Do not jump straight from symptom to patch.

The required chain is:

`Observed behavior → Reproduction → Evidence → Root cause → Regression coverage → Fix → Verification`

A plausible explanation is not enough. When practical, establish the root cause with code-path evidence, a failing test, a minimal reproduction, targeted logging, or another observable signal before changing production code.

## Workflow

### 1. Characterize the defect

Extract:

- expected behavior
- actual behavior
- trigger/action sequence
- affected users or scope if known
- error messages, stack traces, logs, screenshots, or diagnostics supplied by the user
- first known bad version / recent related changes if known

Translate vague reports into a concrete technical statement. For example, distinguish:

- crash vs incorrect result
- intermittent vs deterministic
- UI-only symptom vs deeper application-state defect
- first occurrence vs regression

### 2. Reproduce or build a minimal proof

Search for an existing test or reproduction path first.

Then:

- reproduce the failure using the narrowest relevant path
- capture the observable failure
- identify the relevant state transition and call path
- reduce unrelated variables where possible

If the defect cannot be reproduced, continue with static investigation and available evidence, but explicitly mark the reproduction gap. Do not invent a successful reproduction.

### 3. Investigate systematically

Inspect only the code needed to test plausible hypotheses. Typical areas include:

- recent changes and callers
- state transitions
- nullability and lifetime assumptions
- exception paths
- async/await flow and cancellation
- shared mutable state
- caching
- persistence/serialization
- dependency injection/lifetime configuration
- WPF binding and `DataContext`
- dispatcher/thread affinity
- command `CanExecute` and notification
- collection/property change notifications
- navigation/window lifetime
- resource lookup and XAML loading

For WPF defects, explicitly consider whether the visible symptom is downstream of a state, binding, or thread-affinity problem rather than the UI element itself.

### 4. State the root cause before fixing

Write a compact root-cause statement:

- **Cause:** what is wrong
- **Mechanism:** why it produces the observed behavior
- **Evidence:** what in the code/test/log supports the conclusion

When confidence is low, say so and identify the competing hypotheses rather than presenting a guess as fact.

### 5. Add regression protection

Prefer to encode the defect before applying the fix when the test framework and failure mode make that practical.

A useful regression test should:

- fail for the pre-fix behavior when feasible
- pass for the corrected behavior
- focus on the defect's contract, not incidental implementation

Choose the lowest-level reliable test boundary that captures the defect. For example, a ViewModel/service test is usually preferable to a UI test when it fully proves the failing behavior.

When a test is impractical, document why and use the strongest available validation instead.

### 6. Implement the smallest safe fix

- Change only what is necessary to correct the root cause.
- Preserve unrelated behavior.
- Follow existing architecture and coding patterns.
- Avoid opportunistic refactors unless required to make the fix correct, testable, or safe.
- Do not mask symptoms with broad exception handling, arbitrary retries, sleeps, dispatcher hops, or null suppression.
- For concurrency problems, fix the ownership/order/cancellation issue rather than merely hiding the race.

#### WPF-specific fixes

Pay particular attention to:

- UI-thread affinity of bound state
- `ConfigureAwait` behavior across application layers where relevant
- stale async results updating newer UI state
- cancellation and disposal races
- `INotifyPropertyChanged` / collection notifications
- `DataContext` replacement and object lifetime
- `CommandManager`/command notification behavior used by the repository
- event subscriptions causing leaks or duplicate handlers
- navigation closing a View/ViewModel while async work is still running
- binding path typos that compile successfully but fail at runtime

### 7. Verify the defect and the fix

Run, as applicable:

1. the new/updated regression test
2. relevant unit/integration tests
3. affected project build
4. broader tests when practical
5. targeted runtime or UI validation for WPF-only behavior when needed

For intermittent or timing-sensitive defects, perform repeated or stress-oriented validation when the repository provides a practical mechanism.

Do not claim that the original defect is fixed unless the strongest available evidence supports that conclusion.

### 8. Review for unintended consequences

Inspect the final diff and ask:

- Does the change actually address the stated root cause?
- Could the fix alter behavior on adjacent code paths?
- Did the fix introduce a threading, lifetime, or cancellation problem?
- Did the regression test prove the important boundary?
- Did unrelated files or formatting change?
- Could the bug reappear through another equivalent path?

### 9. Report

Finish with:

- **Root cause**
- **Fix**
- **Regression coverage**
- **Validation performed**
- **Remaining uncertainty / limitations**

Keep the report evidence-based. Distinguish observed facts from assumptions.

## Bug-fixing guardrails

- Do not rewrite the component unless the defect cannot be safely corrected within the existing design.
- Do not close an issue because the symptom disappeared without understanding why.
- Do not add logging as a substitute for analysis when existing evidence is sufficient.
- Do not “fix” a race condition with `Task.Delay`, arbitrary sleeps, or retries unless the intended product behavior explicitly requires them.
- Do not suppress exceptions simply to prevent a crash unless that is the established error-handling contract and the failure is still surfaced appropriately.
- Do not change public APIs, persistence formats, or threading models without considering compatibility and callers.
- When the evidence indicates a deeper architectural problem, fix the defect safely first and clearly identify larger remediation as separate work unless the user explicitly asks for the broader change.
