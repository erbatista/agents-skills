---
name: feature-development
description: Implement new features, user stories, enhancements, commands, views, services, endpoints, and workflow extensions in any codebase, using the repository's existing architecture, patterns, tests, and build conventions. Use whenever the task is to add, build, implement, extend, or support new behavior, even when the word "feature" is not used. Language-agnostic; detects the stack and loads the matching stack skill (dotnet-wpf, blazor) for conventions and validation commands. Do not use when the primary task is diagnosing existing incorrect behavior; use bug-fixing for that.
---

# Feature Development

Deliver a production-ready feature with the smallest coherent change that satisfies the requested behavior and fits the existing architecture.

Precedence when instructions conflict, highest first:

1. explicit user instructions
2. the repository's agent file (`AGENTS.md`, `CLAUDE.md`, or equivalent)
3. the stack skill loaded in step 0
4. this skill

## 0. Identify the stack

Do this before reading any code.

1. If the repository's agent file names a stack skill, load it. That declaration wins over detection.
2. Otherwise inspect the repository root:
   - `*.sln` or `*.csproj` containing `<UseWPF>true</UseWPF>` → load `dotnet-wpf`
   - `*.csproj` alongside `.razor` files → load `blazor`
   - any other stack with no matching skill → continue with this workflow alone
3. Follow the stack skill's "What to read, when" section. It says which reference to open before implementing and before validating.
4. Note the outcome for the report: `Stack skill: dotnet-wpf`, `blazor`, or `none`.

When both WPF and Blazor projects exist in one solution (mid-migration), load the skill for the project that owns the requested behavior; load both only if the feature spans both.

## 1. Understand the request

Extract the desired behavior, acceptance criteria, affected user flow or component, explicit constraints, and likely non-functional requirements (performance, concurrency, persistence, compatibility).

If the request is underspecified, look for established behavior and analogous features in the repository first. Ask only when the missing information materially changes the implementation or makes safe progress impossible; otherwise proceed with a clearly stated assumption.

## 2. Discover the codebase

Before editing:

- read the agent file and any repository-local instructions
- identify the project(s) that own the behavior
- find the closest existing feature with similar UI, command, service, persistence, or validation behavior and use it as the template
- trace the relevant flow end to end through the layers the repository uses
- locate existing tests and test helpers for the affected area

Start with the smallest set of files that establishes the architectural path; expand only when needed. Do not explore the whole repository.

## 3. Plan, then decide whether to pause

Write a short plan: files/components to change, the existing pattern being followed, the main data flow, tests to add or update, risks and assumptions.

Stop and present the plan before editing when any of these apply:

- the change adds a new top-level component: view/page, service, DI registration, persistence entity, or public API surface
- the change touches more files than the threshold in the stack skill's conventions table (default 8)
- the change alters a persistence format, a public API, or a threading model
- an ambiguity in the request would change the design

If the environment cannot ask questions (non-interactive run), end the turn with the plan and no code changes. Otherwise move directly from plan to implementation.

## 4. Implement incrementally

Work in coherent slices. Follow the stack skill's feature checklist for language- and framework-specific rules. Stack-independent rules:

- prefer existing patterns over new abstractions
- keep each responsibility in the layer where the repository already puts it
- preserve existing API contracts, nullability, and error-handling conventions
- add no new dependency, framework, or library when an existing mechanism can satisfy the requirement
- do not refactor or restyle working code that the feature does not require changing

## 5. Add focused tests

Prefer tests that prove behavior over tests that mirror implementation. Choose the smallest useful set from: domain/application/service unit tests, presentation-logic tests (ViewModel, component, controller), integration tests at persistence or external boundaries, and UI tests only if the repository already has a reliable UI-test strategy.

A test should encode an acceptance criterion or guard a meaningful regression boundary. Do not add tests only to raise coverage.

If the repository has no test project, do not create one or introduce test infrastructure unless asked. Validate by other means (step 6) and report the gap.

## 6. Validate

Run the commands in the stack skill's validation reference, in the order it specifies. If no stack skill is loaded, use the commands documented in the repository; if none are documented, use the ecosystem's standard build and test commands and say so.

If a check cannot run in the current environment, state exactly what was not run and why. Never report a check as passed if it did not execute.

## 7. Review the final diff

Inspect the complete diff: remove accidental changes, dead code, debug output, and unused imports/resources; confirm the acceptance criteria are met; check for duplicated logic that should reuse an existing path; verify no unrelated files changed.

## 8. Report

Use this template exactly:

```
## Summary
<what changed, 2-4 sentences>

## Key decision
<the one design choice a reviewer should know about, and the existing pattern it follows>

## Stack skill
<dotnet-wpf | blazor | none>

## Validation
- ran: <commands, each with pass/fail>
- not run: <commands and why> | none

## Tests
<added/updated tests and what each proves> | none (why)

## Assumptions and limitations
<list> | none

## Review focus
<files or areas that deserve the closest look>
```

## Guardrails

- Do not broaden scope because you found unrelated technical debt; mention it in the report instead.
- Do not perform large refactors to make the feature easier unless required for correctness or explicitly requested.
- When several designs are viable, choose the one closest to the surrounding code.
- Preserve backward compatibility unless the request explicitly permits a breaking change.
- If the request mixes a bug and an enhancement, fix the bug with `bug-fixing` first, then return here; keep the two diffs separable.
