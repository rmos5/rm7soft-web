# rm7soft framework: usage and development rules

This is the application-independent guide for using and extending the rm7soft
framework. It assumes no particular business domain, application, palette,
navigation structure or deployment model. A consumer needs the framework
packages, not another application's source code.

Use this document for architecture and development decisions. For project setup,
DI registration and C#/XAML examples, follow the
[application integration walkthrough](create-maui-app.md).

The application template is under revision and is not ready to use. Manual
integration into an ordinary MAUI project is the documented starting point.

## 1. Package responsibilities

| Package | Purpose | Dependency boundary |
| --- | --- | --- |
| `rm7soft.App` | Presentation contexts, commands, messages, validation contracts, editing, localization, workspace state, settings and sessions | Plain .NET; no MAUI, native APIs or concrete database dependency |
| `rm7soft.Maui` | MAUI navigation, dialogs, controls, styles, themes, dispatch and platform adapters | Depends on App; never on a consuming application's types or resources |
| `rm7soft.Storage.Sqlite` | Transactional persistence of keyed application-state documents | Implements App storage contracts; no dependency on MAUI |
| `rm7soft.ChangeTracking` | Property/collection history and grouped changes | Separate package; reused by App instead of reimplemented |

The reviewed framework targets .NET 10, with Android and Windows MAUI targets.
The MAUI library's plain .NET target supports behavior tests without native
handlers. It does not imply support for additional native platforms. Match the
selected package's target frameworks and MAUI dependency versions when creating
an application.

Keep domain entities and operations outside the UI layer. An application may
use its own domain project, database, platform services and settings model. The
framework does not require a particular entity hierarchy, storage technology,
background-processing model or set of pages.

## 2. Compose an application

The application composition root, normally `MauiProgram`, owns registrations.
Use constructor injection in contexts and services. Service-provider access
belongs at composition and UI integration boundaries, not in business methods.

1. Register logging and one language owner as `ITextResources`.
2. Bundle/register text fonts and the Font Awesome font used by shared controls.
3. Merge `SharedStyles`, app styles and app palettes in the application resources.
4. Register one theme owner, with `IThemeState` resolving that same instance.
5. Call `AddRm7softMauiWindowServices()` for scoped navigation and dialogs.
6. Register pages transient and their window-owned contexts scoped.
7. Create a DI scope for each window; attach `MauiWindowContext` before resolving
   its first page. Use `NavigationPage` with `MauiNavigationService`.
8. Initialize application preferences and data, then navigate to the initial page.
9. Dispose the window scope when its window is destroyed. Release separately
   owned operations and subscriptions at their own lifetime boundaries.

The window helper does not register app pages, palettes, language/theme policies,
storage or optional native handlers. These remain explicit application choices.
Register interface aliases with factories returning the existing owner; separate
registrations that construct separate owners are not equivalent.

| State | Default ownership choice |
| --- | --- |
| Applied language and theme | Application singleton |
| Navigation and dialog queues | Window scope |
| Page context and workspace selection | Window scope, unless a shorter lifetime is explicitly implemented |
| Cached page | Its window |
| Edit history and working data | Edit transaction |
| Storage adapter | Application service with its documented concurrency policy |
| Cancellation source and temporary data | Owning operation |

A long-lived service must not capture a shorter-lived context. When a background
service reports progress, pass data through an app-defined boundary and dispatch
UI updates; do not let it retain a window or page accidentally.

## 3. Implement a feature

Use a context for observable state and commands, a view for presentation, and a
domain service for business operations. The context translates user intent into
validated operations and exposes their results to bindings.

| Feature requirement | Framework starting point |
| --- | --- |
| Observable state and validation | `ContextBase` |
| Page feedback and guarded commands | `MessageContextBase` |
| Back-navigation decisions, layout state and refresh | `PageContextBase` |
| Editable properties and history | `EditPageContextBase` |
| Selected items and collection operations | `ItemsPageContext` / `ItemsEditPageContext` |
| Modal interaction | `IUIDialogService` |
| Durable preferences | `ISettingsStore<T>` / `SettingsStore<T>` |
| Durable application-state documents | `IAppStateStore` |
| Session restoration and coalesced saves | `SessionStateCoordinator<TState>` |
| Multiple coordinated panels | Workspace state helpers plus `WorkspaceHostView` |

Do not inherit an editing or workspace context merely to reuse one convenience
method. Use inheritance for a shared lifecycle and composition for independently
varying behavior. Keep page code-behind limited to view/lifecycle integration.

For each feature, define loading, editing, saving, cancellation and navigation
separately. Loading data, selecting a row and refreshing a page must not implicitly
start an edit transaction. Decide how repeated visits reopen a cached context.

### Commands and asynchronous operations

Validate before mutation and before dependent work. Revalidate after an await
when the destination, selection, permissions or eligibility may have changed.
For batch operations, validate the whole batch before changing the destination.

Use the context command guards for user actions and callback guards for events,
native callbacks and deferred work. Await tasks. Guard a deferred callback where
it executes as well as where it is scheduled. Keep technical exceptions in logs
and expose localized messages through the shared message/dialog path.

Cancellation semantics must be explicit. Dialog cancellation returns false/null;
token-aware operations distinguish requested cancellation from failure. Every
operation must restore busy state and release gates in failure/cancellation paths.
Do not add unobserved background tasks to bypass a UI lifetime problem.

Command availability describes whether an action can run. Missing selection,
read-only data and conflicting work can disable it. An application product limit
should normally be explained when the available action is invoked, rather than
silently presenting a disabled control. Domain policy belongs to the application.

### Editing and navigation

Load the working state before starting tracking. Start editing explicitly. Save
through the application's persistence service and accept changes only after a
successful write. Failed saves and canceled navigation retain working values,
selection and undo history.

Route exits through the context's navigation decisions. The MAUI navigation
service manages a stack and page cache; it does not automatically discover and
protect every active edit session. Direct pops, native back controls, window
closing and alternate destinations all need the application's exit policy.

When one logical edit spans several trackers, group the synchronous mutations.
Do asynchronous validation and loading before the group. Use the shared
`EditHistoryPolicy` if history needs limits; do not silently drop undo records.

### Optional capabilities

Use a workspace only when the application needs coordinated panels. Keep panel
identity, selection, scrolling and editing separate. A workspace host presents
consumer data and templates; it does not own domain entities or their disposal.

Use session restoration only for state the application intends to preserve.
Persist stable identifiers and detached DTOs, never controls or services. Define
recovery for missing entities and the precedence of explicit activation requests.
Restoring a session does not imply restoring unsaved edits.

Choose `Native`, `Fade` or `None` for navigation animation. Native timing is
platform-owned; only custom Fade uses the duration/easing options. Configure
transitions through the navigation service rather than adding per-page animation
implementations. A particular duration is a product choice, not a framework rule.

## 4. Presentation rules

**Views describe structure and bindings. Central style dictionaries describe
appearance. Theme dictionaries provide semantic color values.**

Keep colors, typography, spacing, control shapes and visual states in shared
styles and named variants. Layout declarations such as grid rows and binding
paths belong in views; duplicated visual constants do not. Extend existing styles
with `BasedOn` where appropriate so inherited sizing and disabled behavior remain
consistent.

Use semantic keys, not names tied to one application's palette. Put live theme
values behind `DynamicResource`. A stable style reference can use `StaticResource`
while its setters resolve dynamic colors. All palettes must provide the same
required keys with the correct XAML types.

Keep one applied-theme owner. Feed platform theme events into it, and have
consumers observe that owner. Page recreation, forced layout refreshes and
manually assigning colors to individual controls are not substitutes for correct
resource bindings. Preserve the established opacity behavior for disabled states.

Use central Font Awesome glyph definitions and central sizing styles. Select a
control according to its interaction: command, toggle, option cycle or numeric
adjustment. Similar appearance does not make these behaviors interchangeable.
Keep accessibility descriptions localized and test keyboard/focus behavior along
with touch behavior.

Reusable controls expose data, commands, templates and semantic resources. They
must not cast `BindingContext` to an application-specific class, resolve an app
singleton or look up undeclared application resource keys. Publish shared resource
defaults and document the host's override points.

## 5. Localization and encoding rules

Keep one language owner implementing `ITextResources`. Store UI strings in
resources and use live bindings/providers for text that can remain visible while
the language changes. Refresh computed localized properties and option labels
through their owning context; do not create another language service per view.

Framework strings belong in framework resources; domain UI text belongs in the
consumer. A consumer may override framework strings through its own resources.
Document fallback behavior, but do not advertise a language as complete merely
because missing translations resolve to neutral strings.

Maintain the key registry, resource files, typed accessors, translator comments
and formatting placeholders together. Translate each target from the neutral
source, review meaning and terminology, and test key/placeholder completeness.
Never translate user-authored content as a side effect of changing UI language.

Preserve the repository's encoding, BOM and CRLF conventions. Do not run broad
encoding conversion as part of a text change. Resource files must remain valid
XML with correctly escaped text; non-ASCII characters must survive reading,
editing, compilation and display.

## 6. Persistence and lifecycle rules

Persistence adapters own storage mechanics; the app owns defaults, migrations,
document keys and business transactions. Do not assume a keyed JSON state store
is a replacement for a domain repository requiring relational queries.

Capture a coherent save snapshot. Prevent conflicting edits during persistence
or implement explicit version/concurrency handling. An awaited write must not
mark later, unpersisted changes as saved. Define event behavior after a committed
write: a failing notification cannot roll back that write.

Initialization must be idempotent. Page appearance, modal dismissal, application
resume, activity recreation and process startup are different events. Restore
only at the intended boundary; do not replay startup whenever a page appears.

Save meaningful state during normal operation. A final shutdown callback is not
guaranteed. For multiple windows, use distinct session keys when independent
restoration is intended. Cancel/await owned work and detach subscriptions at the
documented boundary, including failed construction and handler replacement.

The framework does not provide application-specific permissions, background
execution, notifications or operating-system integration policies. Implement
those in app/platform adapters without moving them into the plain App layer.

## 7. Extending the framework

### Decide whether behavior belongs in the framework

Keep a feature in the application when it depends on business entities, product
rules, specific routes, stored app preferences or branding. Promote behavior when
its contract is useful across consumers and can operate on explicit inputs,
commands, callbacks or templates without knowing the original application.

Do not create speculative abstractions or an interface for every class. First
identify actual callers, the varying policy and the common lifecycle. A useful
extension has a small public surface and clear ownership.

Choose the owning project:

- Platform-neutral state, decisions and contracts belong in `rm7soft.App`.
- MAUI view behavior, native UI adaptation and presentation belong in `rm7soft.Maui`.
- Concrete storage implementation belongs in its adapter package.
- Domain policy and application orchestration remain with the consumer.

Use the exact `rm7soft` root namespace for framework-owned code. Keep consumer
namespaces separate. Preserve external identifiers when renaming code: persisted
values, resource keys and serialized names can be compatibility contracts.

### Specify the contract before implementation

Every public extension point must document:

| Concern | Required decision |
| --- | --- |
| Inputs and results | Valid values, defaults, nullability and invalid-input behavior |
| Ownership | What it owns, what it borrows, and who disposes it |
| Lifetime | Application, window, context, transaction or operation |
| Threading | Calling thread, dispatch responsibility and notification thread |
| Concurrency | Reentrancy, serialization, coalescing and overlapping calls |
| Cancellation | Whether supported, who controls it, and state left after cancellation |
| Failure | What propagates, what is logged/reported, and whether state was already committed |
| Resources | Required semantic keys/types, defaults and host override points |
| Compatibility | Effects on consumers, derived classes and persisted data |

For example, a reusable collection action should accept its destination and
validation/mutation capabilities explicitly. It should not know a consumer's
entity class, entitlement service or page. Document whether validation completes
before mutation and what happens if an observer fails after mutation.

For a reusable control, define bindable data/commands and a template/style
contract. Attach subscriptions only for the relevant loaded/handler lifetime,
detach on replacement/unload, and guard callbacks against stale owners. Do not
dispose borrowed binding contexts.

### Preserve public and resource contracts

Review public/protected API additions, signatures, defaults, nullability,
inheritance and behavior. Record approved additions in `PublicAPI.Unshipped.txt`;
do not regenerate baselines merely to make diagnostics disappear. MAUI keeps
separate inventories for each target framework.

Treat semantic resource keys, XAML types and style target types as public
contracts too. Update the reviewed resource inventory when necessary, supply
defaults, and test both shared defaults and consumer overrides. A resource that
exists only in one consuming app must not hide a missing framework default.

Follow [API and resource contracts](framework-contracts.md) for baseline details
and [architecture checks](architecture-checks.md) for dependency/naming enforcement.

## 8. Verification and delivery

Test observable contracts, failure paths and user scenarios. Do not add tests
that merely repeat implementation details. Use the smallest verification scope
that covers the change, then expand when failures or affected platforms require it.

| Change | Relevant verification |
| --- | --- |
| App-layer behavior | App tests; cancellation, failures, ordering and disposal |
| Public API, dependency or shared resource | Architecture checks, API inventory review and consumer compatibility |
| MAUI control/style | MAUI tests, defaults/overrides, live theme/language changes, enabled/disabled states |
| Native adapter or layout behavior | Builds for affected targets plus device/emulator checks |
| Storage | Transactions, concurrent access, corrupt/missing state and supported format handling |
| Package | Isolated package consumer, dependency resolution and source/symbol provenance |
| Documentation only | Link/anchor checks, API accuracy and affected example compilation when needed |

From the framework repository, run the affected suites:

```powershell
dotnet test tests/rm7soft.App.Tests/rm7soft.App.Tests.csproj -m:1
dotnet test tests/rm7soft.Maui.Tests/rm7soft.Maui.Tests.csproj -m:1
dotnet test tests/rm7soft.Storage.Sqlite.Tests/rm7soft.Storage.Sqlite.Tests.csproj -m:1
dotnet test tests/rm7soft.Architecture.Tests/rm7soft.Architecture.Tests.csproj -m:1
```

For changes affecting native framework targets:

```powershell
dotnet build src/rm7soft.Maui/rm7soft.Maui.csproj -f net10.0-android -m:1
dotnet build src/rm7soft.Maui/rm7soft.Maui.csproj -f net10.0-windows10.0.19041.0 -m:1
```

Library compilation and tests without native handlers do not establish device
behavior. Keep those verification claims separate. Measure representative startup,
large-list, memory and response-time baselines before claiming an optimization.

Use the repository's package scripts and
[release checklist](package-release-checklist.md) for approved local package
verification and delivery. Check the installed artifact's identity and provenance;
NuGet caches by package ID/version. Keep feed credentials outside source control.
Do not change versioning policy or replace a published payload as an incidental
part of another task.

Demonstrate reuse with independent consumers before promoting an app-specific
implementation into a released framework feature. The repository's
[development rules](rm7soft-development-rules.md) define the concrete acceptance
and workflow requirements; this guide does not relax them. Template verification
and readiness are separate from library verification.

Keep changes focused and logical. Preserve unrelated work and formatting. Local
verification does not authorize a commit, push or publication; follow the
[repository workflow](codex.md) and the scope explicitly approved for the task.

## 9. Review a completed change

Before handing over a feature, establish that its state has one owner, dependencies
point in the correct direction, and cancellation/failure leave usable state.
Confirm navigation cannot bypass pending edits and that successful persistence
is the only point where changes are accepted.

For presentation work, check the central style and semantic resource contract,
all supported palettes, live localization, disabled states, accessibility and
native layout behavior. For framework work, also review compatibility and prove
that a consumer with different data and branding can use the same API.

Report what changed, the behavior it enables, what was actually verified and any
remaining limitations. Keep application-specific design decisions in application
documentation and reusable contracts beside the owning framework library.
