# Feature checklist — .NET / WPF

Read after step 3 (plan) and before step 4 (implement) of `feature-development`.

## C# / .NET

- Preserve nullability annotations and existing API contracts.
- Reuse the DI registrations and service abstractions already present; register new services where the repository registers the others.
- Put domain/application logic in the layer the repository uses for it; keep ViewModels thin.
- Keep `async` all the way through I/O; no `.Result`, `.Wait()`, or `GetAwaiter().GetResult()` on UI paths.
- Propagate `CancellationToken` when the repository does or when the operation can be long-running.
- Preserve exception and error-handling conventions and the user-facing error contract from the conventions table.
- No speculative abstractions; no new NuGet dependency unless the feature requires it.

## WPF / XAML

- Follow the MVVM conventions from the conventions table (toolkit, base classes, command type).
- Respect existing `DataContext`, command, binding, validation, and navigation patterns.
- Keep UI-bound collections and properties on the repository's notification strategy (`INotifyPropertyChanged`, `ObservableCollection`, toolkit-generated properties).
- Do not update bound state from a background thread; marshal using the repository's pattern.
- Consider binding modes, element names, converters, resources, styles, templates, and design-time behavior (`d:DataContext`) only when relevant to the change.
- Preserve keyboard and accessibility behavior (tab order, access keys, automation properties) and existing visual conventions unless the request changes them.
- Prefer existing styles and resource dictionaries; add new resources where the repository already keeps them.

## Runtime issues compilation will not catch — inspect before finishing

- binding path typos and wrong `DataContext` assumptions
- missing resources, styles, or templates (`StaticResource` keys)
- dispatcher / thread-affinity violations
- command enablement (`CanExecute`) not re-evaluated after state changes
- navigation or window lifetime issues (a view closed while async work continues)
- event handlers subscribed on load without a matching unsubscribe

When a runtime check is possible, run with binding errors visible (Output window, or `PresentationTraceSources.TraceLevel=High` on the suspect binding).
