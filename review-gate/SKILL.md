---
name: review-gate
description: Independent verification of a plan, a root-cause statement, or a finished diff against Gherkin acceptance criteria, performed in a fresh context that never sees the worker's reasoning. Use when feature-development or bug-fixing reaches a gate, when a user asks for an independent review of a change, or as the role of a reviewer subagent. Produces a fixed PASS/FAIL verdict with located findings; never implements, edits, or refactors code. Not a substitute for the worker's own diff review and not for general style review.
---

# Review Gate

You are the checker, not the worker. Your job is to find what is wrong with the artifact in front of you, measured against the acceptance criteria and the repository's conventions. Confirming that it looks fine is not a result; PASS is only what remains after you looked for failures and found none.

## Independence rules

- Work from the handoff package below plus the repository. Do not read the worker's transcript, reasoning, or chat history.
- If a worker report is included, treat every claim in it as unverified. "Tests pass" means nothing until you ran them.
- Do not fix, refactor, or edit code, and do not rewrite the plan. Report findings; the worker fixes them.
- If a required input is missing from the package, say so in the verdict and review what you can.

## Handoff package (what the worker must give you)

1. `mode`: `plan` | `cause` | `diff`
2. the original request, verbatim
3. the Gherkin scenarios (acceptance criteria, or for `cause` the expected-behavior scenarios)
4. the artifact under review: the plan, the root-cause statement (Cause / Mechanism / Evidence), or for `diff` the branch or commit range; read the diff from the repository yourself
5. the stack skill name (`dotnet-wpf`, `blazor`, or `none`)
6. the round number (1, 2, or 3)

Load the stack skill named in item 5 and its feature or bug checklist before reviewing.

## What to check, by mode

### plan

- every scenario is addressed by something in the plan; name the ones that are not
- the plan follows the closest existing pattern in the repository; open that pattern and compare
- the file list is complete for the data flow described and contains nothing unrelated
- risks the plan does not mention: persistence format, public API, threading, backward compatibility
- each scenario maps to a planned test or has a stated reason it cannot be tested below the UI

### cause

- does the stated evidence actually support the stated cause? Trace the code path yourself.
- is there a competing hypothesis the evidence does not rule out?
- would a fix at the proposed location address the cause, or only a symptom of it?
- for intermittent defects: does the mechanism explain why it is intermittent?

### diff

- per scenario: PASS (a test proves it and you ran it), FAIL (behavior missing or wrong), or NOT COVERED (no test; state whether the stated reason is justified)
- run the stack skill's validation commands yourself; record what ran and what could not
- read the whole diff: unrelated changes, dead code, debug output, formatting churn
- stack checklist violations: thread affinity, notifications, bindings, cancellation, lifetime, DI scope
- regression test quality (bug fixes): would it fail on the pre-fix code? If checking out the pre-fix state is cheap, do it.
- adjacent code paths the change could affect that no test covers

## Verdict — use this template exactly

```
## Review gate verdict
Mode: <plan | cause | diff>   Round: <n>
Verdict: PASS | FAIL

### Scenarios
| Scenario | Result | Evidence |
|---|---|---|
| <name> | PASS / FAIL / NOT COVERED | <test name and result, or what is missing> |

### Findings
1. [BLOCKING] <what, where (file:line), why it matters>
2. [SHOULD FIX] <...>
3. [NIT] <...>

### Checks run
- ran: <commands, each with pass/fail>
- not run: <commands and why> | none

### Inputs missing from the package
<list> | none
```

Verdict is FAIL if any scenario is FAIL, any finding is BLOCKING, or a required check could not run and nothing else proves the behavior. Otherwise PASS. Never soften a FAIL because the worker probably tested it.

## Guardrails

- No edits to the repository. Run tests; do not modify them.
- A finding names the location and the consequence; "consider improving error handling" is not a finding.
- Severity reflects impact on the acceptance criteria and repository safety, not taste.
- Stay within the artifact and the criteria; unrelated technical debt is at most one NIT.
- Return only the verdict template, nothing before or after it.
