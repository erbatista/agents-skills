---
name: blazor
description: Repository conventions, build/test commands, and framework pitfalls for Blazor codebases - .NET solutions with .razor components, Razor class libraries, Blazor Server or WebAssembly, and interactive render modes. Load whenever working in a Blazor project, and whenever feature-development or bug-fixing routes here in their stack-detection step. Not for WPF or non-.NET projects.
---

# Blazor conventions — STARTER

Same shape as `dotnet-wpf`. This is a starting point: extend it as the Blazor codebase and its conventions settle, and split the checklists into `references/` once they grow.

## Conventions this skill needs — read them from the repository's agent file

Same rule as `dotnet-wpf`: values live only in the repository's `AGENTS.md` / `CLAUDE.md`; this skill stores none. Confirm before implementing; infer from nearby code and flag it in the report when absent.

- Hosting model and render modes in use (Server / WebAssembly / Auto)
- Target framework, solution path, main project paths
- Component base classes and state-management pattern
- DI container and lifetimes (on Server, scoped = per circuit; on WebAssembly, scoped behaves as singleton)
- JS interop conventions
- Test framework, mocking library, and test project paths (bUnit + xUnit/NUnit, or `none`)
- Folder layout
- User-facing error contract
- Build, format, and test commands (defaults in the Validation section below)

Workflow parameter owned by this skill: plan-pause file threshold = 8. The agent file may override it.

## What to read, when

- Feature or bug work → the checklist below (single section until it grows)
- Before any build or test step → the validation section below

## Checklist

Features:

- One component, one responsibility; keep logic in the partial class or a service, not in markup.
- Parameters are inputs; do not mutate `[Parameter]` values inside the component.
- Use `EventCallback` for child-to-parent communication; cascading values only for shared context.
- Async lifecycle methods (`OnInitializedAsync`, `OnParametersSetAsync`) await their work; no blocking waits.
- Call `StateHasChanged` only when state changes outside the normal event/lifecycle path (timers, external events).
- Respect render-mode boundaries: prerendering runs without JS interop; interactive code must not assume it ran during prerender.
- Dispose subscriptions and timers via `IDisposable` / `IAsyncDisposable`.

Bugs — where Blazor defects usually hide:

- UI not updating: state changed on a non-UI path without `StateHasChanged`, or in a component whose parameters did not change
- double execution: `OnInitializedAsync` runs during prerender and again when the component becomes interactive
- JS interop called during prerender, or after the circuit or component is disposed
- DI lifetime confusion between Server and WebAssembly hosting
- stale async results overwriting newer component state
- event-handler re-entrancy and missing `await`
- missing `@key` on rendered lists causing wrong component reuse

## Validation

1. Format / analyzers: `dotnet format <sln> --verify-no-changes`
2. Build: `dotnet build <sln> -c Debug --nologo`
3. Tests: `dotnet test <tests> --no-build` (bUnit tests run on any OS)
4. Runtime check in a browser for rendering, interop, and render-mode behavior, when an environment is available

Blazor projects build and test on any OS; only step 4 needs a runnable app. Never mark a step as passed if it did not execute.
