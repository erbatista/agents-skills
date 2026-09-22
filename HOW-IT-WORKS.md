# How these skills work together

A plain-words guide for the team. No prior knowledge of AI agents needed.

## The idea in one paragraph

We give the AI the same thing we would give a new senior hire: a written process for building a feature, a written process for fixing a bug, the house rules for our WPF codebase, a reviewer who checks the work before it is called done, and a shortcut for our most common job, building a screen for a new backend contract. Each of those is a folder of text files called a *skill*. The AI reads the skill that matches the task and follows it. Nothing here changes what the AI can do; it changes how consistently it does it.

## The three pieces

**A skill** is a folder with a `SKILL.md` file. It describes *how* to do one kind of job, step by step. It knows nothing about any particular repository. Think of it as a runbook.

**The agent file** (`AGENTS.md` in Codex, `CLAUDE.md` in Claude Code) sits at the root of a repository and describes *that repository*: which MVVM toolkit, how to marshal to the UI thread, how to build, how to test, where things live, which feature to copy as the pattern. Think of it as the README you always wished existed, written for the AI.

**The prompt** is what you type for today's task: "add a cancel button to the import dialog."

The AI reads all three. The skill supplies the process, the agent file supplies the local rules, your prompt supplies the goal.

## The six skills

```
  you type a task
        │
        ▼
 ┌─────────────────────┐    contract screen?   ┌──────────────┐
 │ feature-development │◄──────────────────────│ api-feature  │
 │ bug-fixing          │                       └──────────────┘
 └──────────┬──────────┘
            │ step 0: what stack is this repo?
            ▼
 ┌─────────────────────┐
 │ dotnet-wpf / blazor │  house rules, pitfalls, build & test commands
 └──────────┬──────────┘
            │ plan ready ─────────────────► ┌──────────────┐
            │ diff ready ─────────────────► │ review-gate  │  independent PASS / FAIL
            ▼                               └──────────────┘
        final report
```

**feature-development** is the process for anything new: a screen, a command, a service, an enhancement. Understand the request, write down what "done" means, look at how the codebase already does similar things, plan, implement, test, get reviewed, report.

**bug-fixing** is the process for anything broken. Its core rule is "no patching a symptom": reproduce it, find the cause with evidence, write a test that fails, fix, prove the test passes, get reviewed, report. If a request mixes a bug and a change ("update the grid, it's acting weird"), the bug is handled first.

These two are written for any language. They contain no C# or WPF knowledge.

**dotnet-wpf** and **blazor** hold the stack knowledge: the WPF pitfalls (updating bound state from a background thread, binding typos that compile but fail at runtime, commands that never re-enable, stale async results), the `dotnet` commands to build and test, and what to do when the machine can't run WPF. The workflow skills load the right one automatically in their first step by looking at the repository (`.csproj` with `UseWPF` → WPF; `.razor` files → Blazor). They read the repository's conventions from the agent file; they store none themselves, so the same copy works in every repo.

**review-gate** is the independent checker. It runs as a *separate* AI with a fresh memory. It gets the original request, the list of scenarios that define "done," and the plan or the finished change, and nothing else: it never sees how the first AI reasoned or what it claims it tested. It runs the build and tests itself and answers PASS or FAIL with findings that point at a file and line. The worker fixes the findings and asks again, up to three times, then stops and reports what is still open. The worker is forbidden from rewriting the scenarios or weakening a test to get a PASS.

**api-feature** is the shortcut for our most common request: a WPF screen for a new backend contract. You give it a category and a contract name, optionally an existing feature to copy the shape from. It finds the `.proto` and the generated `.cs` under `C:\Dev\ApiSource\<Category>\`, checks that `C:\Dev\Api\ApiContract.dll` already contains the new types (and stops with a clear message if the dll needs a rebuild), reads the contract for its operations and messages, reads the example feature to learn our shape, writes the scenarios, and hands everything to feature-development. It never edits or copies the generated files, and it never invents a type that the dll does not have.

## What a run looks like

A new contract screen:

```
$api-feature category=Kitchen api=rangeOperation  expose only Start and Stop
```

1. api-feature finds `rangeOperation.proto` and `rangeOperation.cs`, confirms the types are in `ApiContract.dll`, lists every operation and message it found, and inventories the example feature named in `AGENTS.md`.
2. It writes the scenarios: for each operation, success, error shown the way the app shows errors, and cancellation.
3. feature-development takes over. Because a new view and a new service are involved, it writes a plan and sends it to review-gate before touching code. You see the plan and the reviewer's verdict together.
4. It implements, writes a test per scenario, runs the build and tests.
5. It sends the finished change to review-gate. If the reviewer finds problems, it fixes them and asks again.
6. You get a report: what changed, the one design decision worth knowing, which stack skill was used, which commands ran and which could not, the scenario-to-test mapping, the gate verdicts, and the contract version it was built against.

A bug:

```
$bug-fixing the timecard grid shows yesterday's rows after changing the filter twice quickly
```

1. It restates the defect precisely and writes the expected behavior as a scenario.
2. It reproduces it and traces the code path, then writes a root-cause statement: cause, mechanism, evidence. If the cause sits in shared code or confidence is low, the statement goes to review-gate first.
3. It writes a test that fails, applies the smallest fix, and shows the test now passes.
4. The change goes to review-gate; the report comes back with the root cause, the fix, the regression test, and anything still uncertain.

## Reading the report

Two lines deserve a glance every time. `Stack skill:` should say `dotnet-wpf` in a WPF repo; `none` means the routing missed and the `AGENTS.md` block is probably missing on that branch. `not run:` lists checks that could not execute and why; that is the AI being honest, not a failure. If the diff gate says `FAIL after 3 rounds`, read the open findings before merging anything.

## Claude Code and Codex: what is the same, what is different

The skill files are identical for both tools. The format is a shared open standard, so one copy serves both; only the folder names, the way you call a skill, and the way the reviewer runs differ.

| | Codex (company repos) | Claude Code (personal projects) |
|---|---|---|
| Skills folder, per repository | `.agents/skills/` | `.claude/skills/` |
| Skills folder, all repositories on your machine | `~/.agents/skills/` | `~/.claude/skills/` |
| Agent file at the repo root | `AGENTS.md` | `CLAUDE.md` |
| Start a task with a skill | `$feature-development ...` | `/feature-development ...` |
| Extra metadata per skill | `agents/openai.yaml` (display name, whether the skill may auto-trigger) | not used; ignored if present |
| Auto-picking a skill from your wording | yes, unless turned off in `openai.yaml` | yes |
| The reviewer (review-gate) | A *custom agent*: copy `codex-agents/review-gate.toml` into `.codex/agents/` (or `~/.codex/agents/`). The workflow spawns it at each gate and prints the handoff package first. Fallback: paste that package into a second session started with `$review-gate` | A *subagent*: copy `claude-agents/review-gate.md` into `.claude/agents/` (or `~/.claude/agents/`). The workflow calls it on its own; you can also say "use the review-gate subagent to check this" |
| Where it runs | Locally on your PC (our standard). A cloud mode exists, but it runs on Linux and cannot run WPF | Locally, in the terminal |
| Reading outside the repo (`C:\Dev\ApiSource`, the example feature) | Needs a sandbox mode that allows it; ours does | Asks permission per folder, or allow it in settings |
| Same skill name in two folders | Both are listed; avoid it | The project copy wins |

Practical consequences:

- Put the company skills in each company repo under `.agents/skills/` and commit them, so a teammate gets them by cloning. For your own projects, one copy in `~/.claude/skills/` is enough; skip `api-feature` there, it is company-specific.
- Type the skill name when you start a task. Both tools can pick a skill on their own from your wording, but that is a judgment call and sometimes misses; typing it never does.
- The reviewer only counts as a reviewer if it runs separately. In both tools that is automatic once the agent definition file is in place (`.claude/agents/` or `.codex/agents/`); until then, it means a second session with the printed package. If neither is possible, the report says `not run` for the gate, and you know the change was not independently checked.
- Skill changes go through pull requests like any other code. Fix a repeated mistake by adding one line: to the skill if the rule would be true in any repo, to `AGENTS.md` if it is about that repo.

## Glossary

- **Skill** — a folder with a `SKILL.md` describing how to do one kind of task.
- **Agent file** — `AGENTS.md` / `CLAUDE.md`; facts about one repository, for the AI.
- **Stack skill** — `dotnet-wpf` or `blazor`; house rules for a technology, loaded automatically.
- **Gate** — a point where a separate AI checks the work and returns PASS or FAIL.
- **Scenario** — one line of "done," in the form Given / When / Then. Tests prove them; the gate checks them.
- **Subagent (Claude Code) / helper agent (Codex)** — a second AI with its own fresh memory, started by the first one to do a bounded job, here the review.
- **Exemplar** — an existing feature we point at and say "build it like this."

## Day-one checklist

1. Clone a company repo; confirm `.agents/skills/` and the `AGENTS.md` block are on your branch (if not: `git checkout main -- .agents/skills AGENTS.md`).
2. Run one small feature with `$feature-development` and read the report's `Stack skill:` line.
3. Run one bug with `$bug-fixing` and read the root-cause statement before the fix.
4. Set up the reviewer for your tool, then re-run one of the above and confirm the report shows a gate verdict instead of `not run`.
