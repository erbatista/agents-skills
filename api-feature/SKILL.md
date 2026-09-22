---
name: api-feature
description: Implement the WPF UI for a new backend contract delivered as a .proto file plus a generated C# class under C:\Dev\Api\<category>\. Invoked as `$api-feature category=<Category> api=<contractName> [ref=<exemplar folder>]` followed by free-text requirements. Resolves the contract files, extracts services, RPCs, messages and enums, inventories an exemplar feature, derives Gherkin scenarios per operation, then hands off to feature-development for the plan gate, implementation, tests and diff gate. Use for any request to build, add, or expose UI for a contract, API, proto, gRPC service, or backend operation. Not for changing the contract itself and not for bug fixes.
---

# API Feature (contract → WPF UI)

Front end to `feature-development` for the team's most common request: a screen for a new backend contract. This skill gathers the inputs, reads the contract and the exemplar, and produces the scenarios and the plan material. `feature-development` does everything after that; do not repeat its steps here.

This skill governs input resolution and contract extraction. `feature-development` governs the workflow from the plan onward, and `dotnet-wpf` plus the target repository's agent file govern conventions.

## Calling convention

```
$api-feature category=<Category> api=<contractName> [ref=<absolute path>] <free text>
```

- `category` — subfolder of `C:\Dev\Api`. Required unless `api` is an absolute path.
- `api` — contract name with or without extension, matched case-insensitively against `<name>.proto` and `<name>.cs` in the category folder.
- `ref` — folder of an existing feature to use as the structural pattern. Optional. Default: the canonical exemplar named in the target repository's agent file; if none is named, the closest feature already in the target repository.
- free text — what the contract cannot say: which operations belong on the screen, the user flow, layout constraints, naming.

A missing required value is asked for, never guessed.

## Boundaries

- `C:\Dev\Api` and the `ref` folder are outside the working directory. Read only; never modify anything under them. All writes go to the current repository.
- If reading those paths fails, the cause is the Codex sandbox (workspace-write mode), not the path. Say so and stop.
- The `.cs` files are generated from the `.proto` by the frontend team's tool and compiled into the API assembly (`api.dll`) that product projects reference. The `.proto` is kept beside them for reference only. Never edit the `.cs`, never hand-write equivalents, never copy either file into the product repository. The new feature uses the contract types through the API assembly reference, exactly as the exemplar does; check the reference form (project reference, package, or binary) and match it.

## 1. Resolve the inputs

1. Confirm `C:\Dev\Api\<category>\` exists. If not, list `C:\Dev\Api` and ask.
2. Find `<api>.proto` and `<api>.cs`, case-insensitively. Both found → proceed. One found → proceed and note it. Neither → list the category folder and ask.
3. Resolve `ref` as above. If the folder does not exist, list its parent and ask.
4. Record for the report: the resolved paths, and the contract version — the commit hash if `C:\Dev\Api` is a git clone (`git -C C:\Dev\Api rev-parse HEAD`), otherwise the contract file's last-modified timestamp.

## 2. Extract the contract

Follow `references/contract-extraction.md`. Read the `.proto` in full; take from the `.cs` only names, via targeted search, never its body. Produce the inventory:

- service name(s) and C# namespace
- each RPC: name, request type, response type, unary or streaming
- each message: fields with type, cardinality, and comments
- enums with values
- the generated client class name

The inventory goes into the plan verbatim so the plan gate can check it against the files.

## 2b. Verify the contract is consumable

Two things can be stale, and a missing type must never be "fixed" by writing it locally.

1. `.proto` vs `.cs`: every service, RPC, and message in the `.proto` must appear in the generated `.cs`. If the `.cs` is missing something, the contract was not regenerated after the backend's last delivery. Stop and report: which names are missing and that regeneration is needed.
2. `.cs` vs the API assembly the product references: find how the target project references the API (project reference, package version, or binary path) and confirm the generated client class and message types are present in that source or version. If they are not, the assembly predates the contract. Stop and report: the reference in use, the contract version from step 1, and that `api.dll` must be regenerated, rebuilt, and the reference updated before the feature can be built.

If a later build fails with a missing contract type, that is the same condition: stop and report, do not define the type in the product repository.

## 3. Inventory the exemplar

Follow `references/exemplar-mapping.md`. List the exemplar's files and classify each (View, ViewModel, model mapping, service wrapper, DI registration, navigation entry, resources, converters, tests). Read the View, ViewModel, service wrapper, and DI registration in full; skim the rest. Then write the mapping table: for each exemplar file, the new file it becomes and what is copied versus derived.

Conflict rule: `ref` supplies structure; the target repository (its agent file and existing features) supplies namespaces, folder layout, DI registration, navigation, and the error contract. The target repository wins.

## 4. Decide the screen scope

Default: one view exposing every RPC of the service, structured like the exemplar's view. The free text overrides this. If the free text names an operation the contract does not contain, ask.

## 5. Write the scenarios

For each exposed RPC, Gherkin scenarios for: success; a server error surfaced through the repository's error contract; cancellation or the in-flight state; and, for streaming RPCs, updates arriving while the view is open. Add one scenario per validation rule visible in the messages (required fields, enum ranges, oneof exclusivity).

## 6. Hand off to feature-development

Continue with `feature-development` from its step 2 (discover), carrying: the request (parameters plus free text), the contract inventory, the exemplar mapping table, and the scenarios. The plan must include the inventory and the mapping table. The plan gate will fire, since a new view, a new service, and a DI registration are always involved; that is intended.

Before the diff gate, search the diff for the exemplar's feature name; none may remain.

In the final report, under "Assumptions and limitations", list the contract paths and version, the exemplar used, and every deviation from the exemplar with its reason.

## Guardrails

- Never invent a field, RPC, or enum value that the `.proto` does not contain.
- Never copy the generated `.cs` or the `.proto` into the product repository, and never define a contract type locally to make a build pass; the API assembly is the only source of contract types.
- Do not change the contract. If it looks wrong or incomplete, report it for the backend team and build what exists.
- Do not reproduce exemplar code the new feature does not need; copy the shape, not the surplus.
