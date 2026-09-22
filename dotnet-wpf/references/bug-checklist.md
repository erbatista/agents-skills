# Bug checklist — .NET / WPF

Read during step 3 (investigate) and step 6 (fix) of `bug-fixing`.

## Where WPF defects usually hide

Investigate these in addition to the stack-independent areas:

- binding and `DataContext`: path typos, wrong `DataContext` after navigation or a template change, `RelativeSource` / `ElementName` resolving to the wrong element
- dispatcher / thread affinity: bound state mutated off the UI thread, `ObservableCollection` modified from a background task
- stale async results: an older request completing after a newer one and overwriting UI state
- cancellation and disposal races: a View/ViewModel closed while a task is still running
- `INotifyPropertyChanged` and collection notifications: missing raises, wrong property name, a replaced collection instance that the view is no longer bound to
- commands: `CanExecute` not re-queried (`CommandManager.InvalidateRequerySuggested` or the toolkit's `NotifyCanExecuteChanged`)
- `ConfigureAwait(false)` in a layer whose continuation touches UI
- event subscriptions: duplicate handlers after repeated navigation; leaks from static or long-lived publishers
- DI lifetimes: a scoped/transient service captured by a singleton; shared mutable state across windows
- resource lookup and XAML loading: `StaticResource` used before the dictionary is merged, implicit style scope
- navigation and window lifetime: `Closing` handlers, owner windows, modal dialogs opened on the wrong thread

Ask explicitly: is the visible symptom downstream of a state, binding, or thread-affinity problem rather than in the element where it shows?

## Fixing WPF defects

- Fix the ownership, ordering, or cancellation problem; do not hide a race with `Task.Delay`, retries, or an extra dispatcher hop.
- A dispatcher call is the right fix only when the code is legitimately off the UI thread and the repository's marshalling pattern is used.
- For stale-result bugs, prefer a `CancellationTokenSource` per request or a request sequence check in the ViewModel over guarding in the View.
- Fix notification bugs at the property or collection, not by forcing a refresh from the View.
- Do not suppress exceptions to stop a crash unless that is the established error contract and the failure is still surfaced to the user or the log.
- Do not change threading models, public APIs, or persistence formats without checking callers.

## Regression tests at the lowest reliable boundary

- ViewModel and service tests can prove most binding-adjacent defects (property raised, collection updated, command state) without loading WPF.
- Keep such tests in a project that does not target `net*-windows` where possible, so they also run in non-Windows environments (see `validation.md`).
- Binding-path and resource defects are usually not unit-testable; document that and validate at runtime.
