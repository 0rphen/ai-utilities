# Boundaries

Data access, structure/DI, and presentation — where each responsibility lives
in an Angular feature. See `SKILL.md` rules 9–12.

## Data layer

Repository port lives in `domain/` (a domain-vocabulary interface, no HTTP
types). Its implementation, the HTTP client usage, the mapper, and error
translation live in `data/`. The repository is the only orchestrator —
nothing else calls a datasource or the HTTP client directly.

`resource()`/`httpResource()` and equivalent query-caching wrappers are a
runtime/presentation concern: they live in `ui/`, wrapping a call to the
repository — never replacing it, never appearing inside `data/`. Cache, retry,
pagination, and polling policy live at the data-access boundary unless they're
genuinely part of application state.

DTOs never cross into `domain/` or `ui/` — map at the `data/`↔`domain`
boundary, both directions.

## Structure and DI

- `domain/`: entities, repository ports, pure business services — no Angular
  import, unit-testable alone.
- `data/`: repository implementation, mapper, HTTP client, datasources.
- `ui/`: smart components, dumb components, optional facade, `resource()`
  wrappers.
- Runtime singletons (interceptors, guards, app-wide store) live wherever the
  project keeps its core/shared split — one instance for the app's lifetime.

Scope each injectable to the lifetime of the state it owns: root-provided for
app-wide singletons, route-scoped for a feature's lifetime, component-scoped
for state that shouldn't outlive one component tree. Avoid interfaces or
injection tokens without a concrete substitution or testability need — a
direct class dependency is enough until one appears.

Naming: follow the installed Angular version's own convention first (check
via `angular-developer`); fall back to `front-guidelines`' naming table only
when no such convention exists.

## Presentation

A **smart** component coordinates: obtains the store/repository/facade, owns
loading/error/empty state, passes plain data down. A **dumb** component
receives inputs, emits outputs, and does nothing else — no `data/` import, no
store as an input, no router/global-state read, no fetch.

Keep templates declarative — no business logic, no derived-value computation,
no method calls that do work rather than read a value. Don't extract a
component unless it earns real reuse or a clearer boundary; premature
extraction adds indirection without payoff.

Change-detection policy is a project-wide decision, declared once: zoneless
where the installed version supports it, `OnPush` otherwise. Detect which
applies rather than assuming.
