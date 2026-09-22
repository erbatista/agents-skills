# Exemplar mapping — copy, derive, conform

Read during step 3 of `api-feature`.

## Inventory the exemplar

1. List every file under `ref` (recursively, excluding `bin`, `obj`, generated code).
2. Classify each file as one of: View (XAML + code-behind), ViewModel, model / DTO mapping, service wrapper (interface + implementation around the generated client), DI registration, navigation or menu entry, resources / styles / templates, converters, validation, tests, other.
3. Read in full: the View, the ViewModel, the service wrapper, the DI registration. Skim the rest.
4. Note how the exemplar's project references the API assembly (project reference, package version, or binary path). The new feature's project references it the same way; if it already does, nothing to add.
5. Note the naming pattern (for example `<Feature>View`, `<Feature>ViewModel`, `I<Feature>Service`, `<Feature>Module`) and the exemplar's feature name; the latter is what the leftover search looks for before the diff gate.

## The three sources, and which one wins

| Concern | Copy from exemplar | Derive from contract | Conform to target repo |
|---|---|---|---|
| View | layout structure, control choices, command wiring, loading and error presentation | which fields and operations appear, labels from proto comments | styles, resource keys, shared templates |
| ViewModel | base class, command type, async and cancellation handling, error handling shape, notification pattern | one property per exposed field, one command per exposed RPC, view state per `oneof` | namespace, folder |
| Service wrapper | interface + implementation shape, cancellation propagation, error mapping, retry policy if any | one method per RPC, request and response mapping | DI lifetime, registration location, logging |
| Models / DTOs | mapping strategy (direct use of generated messages vs mapped DTOs) | fields, types per the type table | naming conventions |
| Navigation | how a view is registered and reached | menu label, route name | navigation pattern |
| Tests | project, fixtures, mocking approach, naming | one test per scenario | test project location |

On conflict between "copy" and "conform", conform wins: the new feature must look like it belongs in the repository it lands in, not in the exemplar's.

## Mapping table to produce

One row per exemplar file:

| Exemplar file | New file | Copied | Derived | Conformed |
|---|---|---|---|---|
| `PatioView.xaml` | `RangeOperationView.xaml` | grid layout, command bindings, busy overlay | fields of `RangeOperationRequest`, buttons per RPC | style keys from the repo's resource dictionary |

Files the new feature does not need get a row with "not needed" and the reason; files the new feature needs but the exemplar lacks (a new converter, a streaming handler) get a row with "no exemplar" and the pattern used instead.

This table goes into the plan, so the plan gate can verify each row against the files.

## Leftover check

Before the diff gate, search the diff for the exemplar's feature name, its namespace, and any string literal or resource key unique to it. Nothing may remain.
