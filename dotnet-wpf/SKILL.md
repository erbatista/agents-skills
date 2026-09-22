---
name: dotnet-wpf
description: Repository conventions, build/test commands, and framework pitfalls for C#/.NET/WPF codebases - solutions with .sln/.csproj and UseWPF, XAML views, ViewModels, MVVM, commands, bindings, and dispatcher/UI-thread work. Load whenever working in a .NET/WPF repository, and whenever feature-development or bug-fixing routes here in their stack-detection step. Not for Blazor, ASP.NET, or non-.NET projects.
---

# .NET / WPF conventions

Stack layer beneath `feature-development` and `bug-fixing`. It answers "how does this repository do X" so the workflow skills never have to guess, and it holds the WPF-specific checklists and validation commands.

## Conventions this skill needs — read them from the repository's agent file

The repository's `AGENTS.md` / `CLAUDE.md` (section "Stack and agent skills") is the only place these values live. This skill stores none of them, so one copy can serve every repository without drifting. Confirm each item there before implementing; if the agent file does not state one, infer it from the closest existing code and list the inference under "Assumptions" in the report.

- Target framework, solution path, main project paths
- MVVM toolkit (CommunityToolkit.Mvvm / Prism / hand-rolled `ViewModelBase` + `RelayCommand`)
- UI-thread marshalling pattern (direct `Dispatcher` call or a dispatcher service)
- DI container, lifetimes, and where registrations live
- Navigation pattern
- Test framework, mocking library, and test project paths (or `none`)
- Folder layout for Views, ViewModels, Services, Models, Converters
- User-facing error contract
- Build, format, and test commands (defaults and environment rules in `references/validation.md`)

Workflow parameter owned by this skill: plan-pause file threshold = 8. The agent file may override it.

## What to read, when

- Implementing a feature → `references/feature-checklist.md`
- Diagnosing or fixing a defect → `references/bug-checklist.md`
- Before any build or test step → `references/validation.md`

Read only what the current task needs.

## Always apply

- Keep `async` all the way through I/O boundaries; never introduce `.Result`, `.Wait()`, or `GetAwaiter().GetResult()` on UI paths.
- Never update WPF-bound state from a background thread; marshal with the pattern in the table above.
- Propagate `CancellationToken` where the repository already does or where the operation can be long-running.
- Business rules belong in domain/application code, not in ViewModels, and never in code-behind unless the repository explicitly uses code-behind for that concern.
- Binding paths, resource keys, and `DataContext` assumptions compile silently and fail at runtime; treat them as correctness, not styling.
- Reuse the toolkit, container, and navigation pattern in the table; do not introduce a second one.
