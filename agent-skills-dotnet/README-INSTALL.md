# Installing these skills

## What is in this package

- `feature-development/` — generic feature workflow (any language)
- `bug-fixing/` — generic defect workflow (any language)
- `dotnet-wpf/` — stack layer for C#/.NET/WPF: conventions table, feature and bug checklists, validation commands
- `blazor/` — starter stack layer for Blazor, same shape
- `AGENTS-snippet.md` — block to paste into each repository's `AGENTS.md` / `CLAUDE.md`
- `README-INSTALL.md` — this file

## Where to put them — pick a scope

The skills are written to work across repositories: the workflow skills are language-agnostic, and repository-specific values live in each repo's agent file, not in the skill.

| Tool | Scope | Location | Use when |
|---|---|---|---|
| Codex | Repository | `<repo>/.agents/skills/` (commit it) | Team repos; anyone who clones gets them. Required for Codex cloud tasks, which only see what is in the repo. |
| Codex | User | `~/.agents/skills/` | Your own machine, every repo you open locally. |
| Codex | Admin | `/etc/codex/skills/` | Managed developer machines or shared containers. |
| Claude Code | Personal | `~/.claude/skills/` | All your projects. |
| Claude Code | Project | `<repo>/.claude/skills/` (commit it) | One repo, shared with collaborators. |

Recommended setup:

- **Company (Codex):** keep the four folders in a small internal git repo (`agent-skills`). Commit a copy into `.agents/skills/` of each product repo so onboarding is `git clone`, and add the `AGENTS-snippet.md` block to that repo's `AGENTS.md` with its values. Update the source repo, then sync copies.
- **Personal (Claude Code):** copy the four folders once into `~/.claude/skills/`. Add the snippet block to each project's `CLAUDE.md`; a project without one gets inferred conventions, flagged in the report.
- Codex follows symlinks, so `~/.agents/skills/` can be a symlink into your `agent-skills` checkout instead of a copy.

Do not install the same skill name at two scopes in Codex; it lists both instead of merging them. Folder names must match the `name` field in each `SKILL.md`; do not rename them. `AGENTS-snippet.md` and this README are not skills; keep them out of the skills folder. Codex and Claude Code pick up new or changed skills automatically; restart if one does not appear.

The tree at either location should look like:

```
skills/
  feature-development/SKILL.md
  bug-fixing/SKILL.md
  dotnet-wpf/SKILL.md
  dotnet-wpf/references/feature-checklist.md
  dotnet-wpf/references/bug-checklist.md
  dotnet-wpf/references/validation.md
  blazor/SKILL.md
```

## Before first use

Nothing to fill in inside the skill folders; repository facts live only in each repository's agent file, so the same skill copy serves every repo.

Per repository:

1. Paste the `AGENTS-snippet.md` block into that repo's `AGENTS.md` (and/or `CLAUDE.md`) and fill in the values and commands.
2. Delete `blazor/` from the skills folder if nothing in that repo uses Blazor.

A repository without the block still works: the agent infers conventions from nearby code and lists the inferences in its report.
Filling the block in is what turns "infers" into "follows the standard".

## How the routing works

Each workflow skill starts with step 0:

1. If the agent file names a stack skill, load it.
2. Otherwise detect from the repo root: `.sln`/`.csproj` with `<UseWPF>true</UseWPF>` → `dotnet-wpf`; `.csproj` plus `.razor` files → `blazor`; anything else → generic workflow only.
3. The stack skill's "What to read, when" section says which reference to open and when.

Every report ends with a `Stack skill:` line, so a missed dispatch is visible immediately.

## Test the routing once

Run two prompts and check which files the agent opened:

- In the WPF repo: "Add a cancel button to the order import dialog" → should load `feature-development`, then `dotnet-wpf` and its `feature-checklist.md`, and report `Stack skill: dotnet-wpf`.
- In a non-.NET folder: the same prompt → should load `feature-development` only and report `Stack skill: none`.

## Invoking explicitly

Codex: `$feature-development <task>` or `$bug-fixing <task>`. Claude Code: `/feature-development` or `/bug-fixing`.
Implicit invocation from the description also works but is less reliable; for onboarding, the explicit form is the safe habit.
