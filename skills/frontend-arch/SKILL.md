---
name: frontend-arch
description: Framework-agnostic frontend architecture rules — layering and dependency direction, feature-first folder structure, atomic-design component decomposition, server/client state placement, and clean-code principles applied inside each layer (naming, function size, SRP, DRY/KISS, error handling, testability, SOLID). Use when scaffolding a new frontend project, structuring or refactoring features, deciding where a file/component/piece of state belongs, splitting an oversized component, or reviewing/auditing a frontend codebase for architecture boundary violations or code-quality issues — in any framework (React, Angular, Vue, Svelte, Solid, or none).
allowed-tools: Read, Write, Edit, Glob, Grep
---

# Frontend Architecture Guidelines

Not a framework, not a starter template, not a specific state library — a placement and dependency rule set for where code goes and which direction it's allowed to depend, applicable to any component-based frontend regardless of runtime.

## Overview

A frontend codebase rots the same way regardless of framework: business rules leak into components, a `shared/` folder becomes a dumping ground nothing can be safely deleted from, one API response shape ripples into components three features away, and a single screen grows past 500 lines because nothing tells it where to stop. These are architecture problems, not framework problems — the fix is the same whether the components are written with hooks, signals, or observables.

This skill gives that fix as two orthogonal decompositions. Vertically: domain → application → infrastructure → presentation, with dependencies pointing inward only. Horizontally, the atomic ladder — atoms → molecules → organisms → templates → pages — crossed with a smart/dumb axis; its reusable rungs (atoms, molecules) live in a shared design system outside any single feature, while organisms/templates/pages live inside each feature's presentation layer. Neither decomposition is a mandate to build all of it — the point is to know where a boundary *would* go, then apply as much of it as the project's actual complexity earns. Placement alone doesn't finish the job, either: a file sitting in the right folder can still be unreadable, untestable, and quietly coupled to three other files — the code inside every layer needs its own discipline.

## Instructions

1. **Adopt before imposing**: read the existing structure before changing anything — naming, folder depth, how state is currently split. Match it. If it conflicts with a rule below in a way that matters, say so and ask or note the tradeoff rather than silently restructuring an unrelated part of the codebase.
2. **Scale the architecture to the project**: the four layers below are a ceiling, not a floor. A CRUD screen with no business rules doesn't need a use case class — a thin service call is enough. A domain with no invariants to protect doesn't need an entity class — a plain interface is enough. Introduce a layer only once something in the project actually needs the seam it provides.
3. **Dependencies point inward, always**: domain depends on nothing else in the app; application depends only on domain; infrastructure and presentation may depend on domain and application, never the reverse. This is the one rule that doesn't scale down — violate it once and every layer below it stops being trustworthy. See `references/layers.md`.
4. **Keep the domain framework-free**: no import of a UI framework, HTTP client, router, or storage API inside domain or application code. If a domain type can't be unit-tested without mounting a component or spinning up a DI container, something crossed the boundary. See `references/layers.md`.
5. **Invert I/O through ports**: the inner layer declares the interface it needs (a repository, a gateway); the outer layer implements it. Domain code calls `ProductRepository.findById()`, never `fetch()` or a generated API client directly.
6. **Cross the boundary through use cases**: presentation calls into application-layer use cases, never straight into a repository or an HTTP client. This is the seam that keeps a component testable with a fake and keeps business rules out of templates.
7. **Map DTOs to entities at the edge**: the wire shape (whatever an API actually returns) gets translated into a domain entity in infrastructure, at the mapper, and never referenced again past that point. A component prop typed as the raw API response is a boundary violation.
8. **Organize by feature, then by layer**: `features/<feature>/{domain,application,infrastructure,presentation}`, not global `services/`, `models/`, `components/` buckets holding every feature's files interleaved. A feature should be deletable by deleting one folder. See `references/structure.md`.
9. **Decompose presentation by atomic level and by smartness**: atoms and molecules are dumb, reusable, and live in the shared design system, never inside a feature — the test is the prop, not a feeling, since a component needing a domain entity as a prop is an organism regardless of size. Organisms are self-contained widgets, templates are pure layout scaffolding with no logic or injected dependencies, pages are smart and own the use-case calls, and all three live inside their feature's presentation folder. Atomic level and smartness are two different axes — a component can be small (atomic level) and still be smart, or large (organism) and still be dumb. See `references/presentation.md`.
10. **Separate server state from client state**: server state (fetched, cached, can go stale, owned by a use case) and client/UI state (a toggle, a form draft, a selected tab) have different lifecycles and don't belong in the same store or the same hook. See `references/state.md`.
11. **Name files by role, not by folder position alone**: a suffix (`.entity`, `.usecase`, `.repository`, `.dto`, `.mapper`, `.port`, `.store`) should make a file's layer identifiable from its name in a flat search result. See `references/structure.md`.
12. **Enforce boundaries mechanically**: a dependency-direction rule that only a human remembers to check gets violated within a month. Wire an import-boundary lint rule (e.g. `eslint-plugin-boundaries`, `import/no-restricted-paths`) so a domain file importing from infrastructure fails the build, not just the review. See `references/structure.md`.
13. **Write clean code inside every layer, not just at its boundary**: meaningful names, small single-purpose functions, no magic numbers or strings, explicit error handling, and dependencies injected rather than reached for inline — these apply as much inside a use case or an organism as they do at the seam between layers. A correctly-placed file with a 300-line function and three swallowed `catch` blocks is not clean code. See `references/clean-code.md`.
14. **In review mode**, walk `references/antipatterns.md` and report each match as `file:line → antipattern → concrete replacement`.

## Best Practices

1. **Colocate tests with the unit they test**: `product.usecase.ts` and `product.usecase.spec.ts` in the same folder, not a parallel `__tests__` tree that drifts out of sync.
2. **One public entry per feature**: a feature's `index` re-exports what other features are allowed to use; nothing outside the feature imports past that entry into its internals.
3. **No barrel file that creates a cycle**: a barrel that re-exports something which itself imports the barrel is a circular import waiting to surface. Import the concrete module instead.
4. **Composition over a base-component inheritance chain**: extend behavior by wrapping or slotting components, not by growing a `BaseCard extends BaseWidget extends BaseComponent` hierarchy.
5. **Templates stay styling-only, pages stay logic-only**: refactoring a smart component's markup into a template is a mechanical cut-and-paste — if it isn't, logic leaked into the template.
6. **A shared module earns its place on the third consumer — except the design system**: this governs promoting a feature's own code (an organism, a util, a hook) to `shared/` once a second unrelated feature needs it, not preemptively "in case". It doesn't apply to atoms and molecules: a primitive with no domain vocabulary has nothing to be local to, so it starts in `shared/ui` from the moment it's written.
7. **Keep async boundaries at the container**: an atom or molecule takes data and callbacks as props/inputs; it doesn't call a use case or subscribe to a store itself.

## Troubleshooting

### A circular import appears right after adding a barrel file
The barrel re-exports a module that itself imports something from the barrel — a cycle that didn't exist when each file was imported directly. **Fix**: import the concrete file, not the barrel, for any in-feature or in-layer import; reserve the barrel for consumers outside the feature.

### A domain unit test suddenly needs a DI container or component harness to run
A domain or application class picked up a framework-specific injected dependency (an HTTP client, a router, a framework service) instead of a plain interface. **Fix**: depend on the port interface and inject a hand-written fake in the test — a domain test that needs the framework's test harness has a framework import somewhere it shouldn't.

### A "shared" folder has become a dumping ground nothing can be removed from
Nothing was ever promoted there on a rule — things landed there because no one wanted to decide where they belonged, and now everything imports from it so nothing is provably dead. **Fix**: for each file, find its actual consumers; anything with exactly one feature consumer moves back into that feature.

### A change to one API response breaks components in unrelated features
The DTO shape is being passed straight through to components instead of being mapped to a domain entity at the infrastructure boundary — so every consumer is implicitly coupled to the wire format. **Fix**: add the mapper at the edge, type the use case's return value as the entity, and update the components to depend on that instead.

## Constraints and Warnings

- **The dependency direction never inverts** — domain and application code never import from infrastructure or presentation, regardless of project size.
- **No framework import inside domain or application** — no component, no DI decorator, no HTTP client, no router, no storage API.
- **No DTO type used as a component prop or use-case return type** — map to an entity at the infrastructure boundary first.
- **No business rule expressed inside a template or JSX/markup conditional** — a rule that needs to be tested belongs in domain or application code, not markup.
- **No deep import into another feature's internals** — go through that feature's public entry, or the dependency isn't visible from a flat search of imports.

## References

- **[references/layers.md](references/layers.md)** — the dependency rule, what belongs in domain/application/infrastructure, a worked end-to-end trace across all four layers, and the collapse table for scaling layers down by project size. Open before scaffolding a new feature or arguing about where a file belongs.
- **[references/structure.md](references/structure.md)** — the feature-first folder tree, file-suffix naming scheme, barrel-file policy, and an import-boundary lint config sketch. Open before creating folders or wiring lint enforcement.
- **[references/presentation.md](references/presentation.md)** — the atomic-design levels crossed with the smart/dumb axis, the templates-have-no-logic rule, and organisms-as-widgets. Open before decomposing or splitting a UI component.
- **[references/state.md](references/state.md)** — server state vs. client state vs. URL state, where each is owned, and loading/error/empty boundary placement. Open before adding a store or deciding where a piece of state lives.
- **[references/clean-code.md](references/clean-code.md)** — naming, function size, SRP inside a layer, DRY/KISS, error handling, testability, and SOLID as it complements the architecture. Open before writing or reviewing the code inside any layer, not just deciding where it goes.
- **[references/antipatterns.md](references/antipatterns.md)** — the antipattern → replacement table used in review/audit mode.
