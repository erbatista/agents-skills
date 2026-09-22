# Validation — .NET / WPF

Read before any build or test step of either workflow skill.

## Commands

Use the commands in the repository's agent file. If it defines none, use these defaults with the repository's paths:

1. Format / analyzers: `dotnet format <sln> --verify-no-changes`
2. Build: `dotnet build <sln> -c Debug --nologo`
3. Targeted tests: `dotnet test <tests> --no-build --filter "FullyQualifiedName~<Namespace.Or.Class>"`
4. Broader suite (when practical): `dotnet test <sln> --no-build`
5. WPF runtime inspection: launch the affected view and check bindings, resources, command state, and threading (Windows only)

Run 1–3 always; 4 when the change touches shared code; 5 for any XAML or ViewModel change when a Windows environment is available.

## Environment rules

- **Windows** (Codex CLI / IDE on a developer machine): all steps apply.
- **Non-Windows** (Codex cloud, Linux containers, Linux CI):
  - WPF projects compile only with `-p:EnableWindowsTargeting=true`; add it to the build and test commands.
  - Test projects targeting `net*-windows` will not execute. Run only platform-neutral test projects.
  - Report step 5 and any skipped test project under "not run" with the reason.
- Never mark a step as passed if it did not execute. If a command fails for environment reasons rather than code reasons, say which.

## No test project

Do not scaffold a test project, test infrastructure, or CI changes unless asked. Validate with steps 1, 2, and 5 plus targeted static inspection of the changed code path, and report the gap so the team can decide.
