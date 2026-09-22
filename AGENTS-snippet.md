# Paste the block below into each repository's AGENTS.md (Codex) and/or CLAUDE.md (Claude Code), then fill in the values.
# The dotnet-wpf skill reads these values from here and stores none of its own; a row you leave out is inferred from code and flagged in the report.

## Stack and agent skills

This repository is a .NET <version> / WPF solution (`<path/to/solution.sln>`).

- For any feature, enhancement, or new behavior, use the `feature-development` skill.
- For any defect, crash, regression, or incorrect behavior, use the `bug-fixing` skill.
- Both must load the `dotnet-wpf` skill. The table below is the repository's source of truth for its conventions.
- Every change passes the `review-gate` skill (diff mode) in a separate agent before the final report.
- For UI over a new backend contract, use `$api-feature category=<Category> api=<contract>`; it hands off to `feature-development`.

| Item | Value |
|---|---|
| Target framework | <net8.0-windows> |
| Solution / main projects | <paths> |
| MVVM toolkit | <CommunityToolkit.Mvvm / Prism / hand-rolled> |
| UI-thread marshalling | <pattern or service name> |
| DI container and lifetimes | <container; where registrations live> |
| Navigation pattern | <...> |
| Test framework + mocking | <xUnit + NSubstitute, ...> |
| Test project(s) | <paths, or none> |
| Folder layout | <where Views, ViewModels, Services, Models live> |
| User-facing error contract | <...> |
| Plan-pause file threshold | 8 |
| Canonical exemplar feature (for `api-feature`) | <absolute path, e.g. C:\Dev\Patio, or a folder in this repo> |
| API assembly reference | C:\Dev\Api\ApiContract.dll (binary reference; note the hint path form used in this repo's .csproj files) |

Validation commands:

- format: `dotnet format <path/to/solution.sln> --verify-no-changes`
- build:  `dotnet build <path/to/solution.sln> -c Debug`
- tests:  `dotnet test <path/to/tests.csproj> --no-build`

If a task mixes a bug and an enhancement, fix the bug first, then add the enhancement, as separable changes.
