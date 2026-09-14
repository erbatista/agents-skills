---
name: feature-development
description: Implement new C#/.NET/WPF features using the repository's existing architecture, patterns, tests, and build conventions. Use for new capabilities, user stories, enhancements, commands, views, ViewModels, services, workflows, and feature extensions; do not use for defect-first investigations where the primary task is diagnosing existing incorrect behavior.
---

# Feature Development

## Purpose

Deliver a production-ready feature with the smallest coherent change that satisfies the requested behavior and fits the existing C#/.NET/WPF architecture.

Treat repository conventions, `AGENTS.md`, solution/project structure, existing tests, and existing patterns as the source of truth. User instructions take precedence over this skill when they conflict.

## Operating principles

- Understand the repository before editing code.
- Prefer existing patterns over inventing new abstractions.
- Make the smallest coherent change; avoid unrelated refactors.
- Keep View, ViewModel, domain/application logic, and infrastructure responsibilities aligned with the existing architecture.
- Treat UI-threading, async behavior, cancellation, lifetime, and binding semantics as part of correctness in WPF.
- Add or update meaningful automated tests for behavior that can be tested below the UI layer.
- Validate with the repository's real build/test commands and report evidence.
- Do not claim success for checks that were not actually run.

## Workflow

### 1. Understand the request

Extract:

- desired behavior
- acceptance criteria
- affected user flow or component
- explicit constraints
- likely non-functional requirements such as performance, threading, persistence, or compatibility

If the request is underspecified, inspect the repository for established behavior and analogous features first. Ask a question only when the missing information materially changes the implementation or makes safe progress impossible; otherwise proceed with a clearly stated assumption.

### 2. Discover the codebase

Before editing:

- Read applicable `AGENTS.md` files and repository-local instructions.
- Identify the solution/project(s) that own the behavior.
- Find the closest existing feature with similar UI, command, service, persistence, or validation behavior.
- Trace the relevant flow from WPF View/XAML through ViewModel into application/domain/infrastructure code as applicable.
- Locate existing tests and test helpers for the affected behavior.
- Inspect dependency injection, navigation, resource dictionaries, converters, command patterns, and state-management conventions only as relevant to the feature.

Do not explore the entire repository indiscriminately. Start with the smallest set of files that can establish the architectural path, then expand only when needed.

### 3. Plan before implementation

Create a concise implementation plan that names:

- files/components likely to change
- the existing pattern to follow
- the main behavior/data flow
- tests to add or update
- notable risks or assumptions

For non-trivial changes, wait for the plan to be reviewed before making broad edits when the surrounding workflow requires human approval. For ordinary execution, move directly from plan to implementation without unnecessary ceremony.

### 4. Implement incrementally

Implement the feature in coherent slices.

#### C#/.NET

- Preserve nullability and existing API contracts.
- Reuse dependency injection and service abstractions already present.
- Prefer clear domain/application logic over putting business rules in ViewModels.
- Preserve async all the way through I/O boundaries; do not introduce `.Result`, `.Wait()`, or blocking waits in UI paths.
- Propagate `CancellationToken` when the repository already uses cancellation or the operation can be long-running.
- Preserve exception/error-handling conventions and user-facing error behavior.
- Avoid speculative abstractions or new dependencies unless the feature genuinely requires them.

#### WPF/XAML

- Follow the repository's MVVM conventions.
- Keep business logic out of code-behind unless the repository explicitly uses code-behind for that concern.
- Respect existing `DataContext`, command, binding, validation, and navigation patterns.
- Keep UI-bound collections and properties consistent with the application's established notification strategy.
- Do not update WPF-bound state from a background thread; marshal to the UI thread using the repository's established pattern when needed.
- Consider binding modes, element names, converters, resources, styles, templates, and design-time behavior only when relevant to the change.
- Preserve keyboard/accessibility behavior and existing visual conventions unless the request explicitly changes them.

### 5. Add focused tests

Prefer tests that prove behavior, not implementation details.

For a feature, consider the smallest useful set of:

- application/domain/service unit tests
- ViewModel behavior tests
- integration tests for persistence or external boundaries
- UI tests only when the repository already has a reliable UI-test strategy and the behavior cannot be meaningfully covered lower in the stack

Do not manufacture tests solely to increase line coverage. A test should encode an important acceptance criterion or protect a meaningful regression boundary.

### 6. Validate

At minimum, run the repository's applicable validation commands, typically:

1. format/analyzer checks if required by the repo
2. build of affected project(s) or solution
3. relevant tests
4. broader test suite when practical and appropriate

For WPF changes, also inspect for likely runtime issues that compilation will not catch, such as:

- incorrect binding paths
- missing resources/styles/templates
- invalid `DataContext` assumptions
- dispatcher/thread-affinity problems
- command enablement/state transitions
- navigation/lifetime issues

If a required check cannot run, state exactly what was not run and why.

### 7. Review the final diff

Before finishing:

- inspect the complete diff
- remove accidental changes, dead code, debug output, and unused usings/resources
- confirm the implementation matches the acceptance criteria
- check that tests cover the important behavior
- check API and architectural boundaries
- look for duplicated logic that should reuse an existing path
- verify no unrelated files changed

### 8. Report

Finish with a concise report containing:

- What changed
- Main design/implementation decision
- Tests/builds/checks actually run
- Any assumptions or known limitations
- Files or areas the reviewer should pay particular attention to

## Feature-specific guardrails

- Do not silently broaden scope because you discover unrelated technical debt.
- Do not perform large refactors merely to make the new feature easier unless the refactor is necessary for correctness or explicitly requested.
- Do not introduce a new WPF framework, MVVM toolkit, DI container, state-management library, or persistence technology when an existing repository mechanism can satisfy the requirement.
- Do not rewrite working XAML or code simply to make it stylistically different.
- When multiple designs are viable, prefer the one that best matches surrounding code and minimizes long-term surprise.
- Preserve backward compatibility unless the requirement explicitly permits a breaking change.
