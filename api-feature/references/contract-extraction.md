# Contract extraction — .proto plus generated C#

Read during step 2 of `api-feature`.

## The `.proto` is the source of meaning

Read it in full. Extract, in this order:

- `package`, and `option csharp_namespace` if present (the namespace the generated types live in)
- every `service` block: each `rpc Name (Request) returns (Response)`, noting `stream` on either side
- every `message`: each field's type, name, number, and modifiers (`repeated`, `optional`, `oneof` group, `map<K,V>`)
- every `enum` and its values, including the zero value
- `import` lines, which tell you which shared messages come from another contract (typically `Common`)
- comments (`//`, `/* */`) on services, RPCs, and fields: they usually carry the business meaning and validation rules

## The `.cs` supplies exact names only

It is generated (look for `DebuggerNonUserCodeAttribute` and the header comment). Do not open it in full; its size wastes context and its body is never edited. Search for:

- `namespace ` — the actual namespace
- `public static partial class ` — the service container class
- `Client : grpc::ClientBase` — the generated client class to wrap
- `public virtual` methods ending in `Async(` — the callable operations and their exact signatures
- `public sealed partial class ` — the message type names

If a name in the `.cs` differs from the `.proto` (casing, pluralization), the `.cs` name is the one the code must use.

## Type mapping for WPF

| proto | Model / ViewModel | Notes |
|---|---|---|
| `string` | `string` | empty string, not null, is the proto default |
| `int32` / `int64` | `int` / `long` | |
| `float` / `double` | `float` / `double` | |
| `bool` | `bool` | |
| `bytes` | `byte[]` | |
| `google.protobuf.Timestamp` | `DateTimeOffset` (or the repository's convention) | convert at the service boundary, never in XAML |
| `google.protobuf.Duration` | `TimeSpan` | |
| wrapper types (`StringValue`, `Int32Value`, …) and `optional` fields | nullable | presence matters; bind with that in mind |
| `repeated T` | `ObservableCollection<T>` when the UI edits it, otherwise `IReadOnlyList<T>` | follow the exemplar |
| `map<K,V>` | `Dictionary<K,V>` or a list of pairs for binding | |
| `enum` | C# enum from the generated code, plus a converter or display list for the UI | the zero value is often "unspecified"; hide or validate it |
| `oneof` | mutually exclusive view state (one property set, others null) | a scenario per branch |

Streaming RPCs: server-streaming becomes an async loop that marshals each item to the UI thread and honors cancellation on view close; client-streaming and bidirectional are rare and warrant a scenario each.

## Never

- edit, copy, or hand-write equivalents of generated types; contract types come only from the referenced API assembly
- proceed past a name that is in the `.proto` but not in the `.cs` or the referenced assembly; that is a regeneration or rebuild gap to report, not a gap to fill
- infer a field the `.proto` does not declare, however obvious it seems
- treat a proto comment as a validation rule the UI must enforce unless the request or the exemplar does the same
