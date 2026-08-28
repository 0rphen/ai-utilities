---
name: frontend-arch
description: Framework-agnostic frontend architecture rules — layering and dependency direction, feature-first folder structure, atomic-design component decomposition, server/client state placement, and clean-code principles applied inside each layer (naming, function size, SRP, DRY/KISS, error handling, testability, SOLID). Use when scaffolding a new frontend project, structuring or refactoring features, deciding where a file/component/piece of state belongs, splitting an oversized component, or reviewing/auditing a frontend codebase for architecture boundary violations or code-quality issues — in any framework (React, Angular, Vue, Svelte, Solid, or none).
allowed-tools: Read, Write, Edit, Glob, Grep
---

# Frontend Architecture Guidelines

Not a framework, not a starter template, not a specific state library — a placement and dependency rule set for where code goes and which direction it's allowed to depend, applicable to any component-based frontend regardless of runtime.

## Overview

A frontend codebase rots the same way regardless of framework: business rules leak into components, `shared/` becomes a dumping ground, one API response shape ripples into unrelated features, a screen grows past 500 lines with no seam telling it where to stop. This skill gives two orthogonal decompositions to fix that — vertical (domain/application/infrastructure/presentation) and horizontal (atomic-design × smart/dumb) — applied only as far as the project's actual complexity earns, plus the clean-code discipline needed inside each layer once placement is right.

## Instructions

1. **Adopt before imposing**: read the existing structure — naming, folder depth, state split — before changing anything. Match it; if it conflicts with a rule below, say so rather than silently restructuring.
2. **Scale the architecture to the project**: the four layers below are a ceiling, not a floor. Introduce one only once something in the project needs the seam it provides.
3. **Dependencies point inward, always**: domain depends on nothing else in the app; application depends only on domain; infrastructure and presentation may depend on domain and application, never the reverse. See `references/layers.md`.
4. **Keep the domain framework-free**: no UI framework, HTTP client, router, or storage API import inside domain or application code. See `references/layers.md`.
5. **Invert I/O through ports**: the inner layer declares the interface it needs; the outer layer implements it.
6. **Cross the boundary through use cases**: presentation calls application-layer use cases, never a repository or HTTP client directly.
7. **Map DTOs to entities at the edge**: translate the wire shape into a domain entity in infrastructure, at the mapper, and never reference the DTO again past that point.
8. **Organize by feature, then by layer**: `features/<feature>/{domain,application,infrastructure,presentation}`, not global `services/`, `models/`, `components/` buckets. See `references/structure.md`.
9. **Decompose presentation by atomic level and by smartness**: atoms/molecules are dumb, reusable, shared-only; organisms are self-contained widgets; templates are pure layout; pages are smart and own use-case calls. The two axes are independent. See `references/presentation.md`.
10. **Separate server state from client state**: different lifecycles, never the same store or hook. See `references/state.md`.
11. **Name files by role**: a suffix (`.entity`, `.usecase`, `.repository`, `.dto`, `.mapper`, `.port`, `.store`) should make a file's layer identifiable from a flat search result. See `references/structure.md`.
12. **Enforce boundaries mechanically**: wire an import-boundary lint rule (`eslint-plugin-boundaries`, `import/no-restricted-paths`) so a violation fails the build, not just the review. See `references/structure.md`.
13. **Write clean code inside every layer, not just at its boundary**: meaningful names, small single-purpose functions, no magic numbers/strings, explicit error handling, injected dependencies. See `references/clean-code.md`.
14. **In review mode**, walk `references/antipatterns.md` and report each match as `file:line → antipattern → concrete replacement`.

## Best Practices

1. **Colocate tests with the unit they test** — see `references/structure.md`.
2. **One public entry per feature**: only its `index` is imported from outside.
3. **No barrel importing from its own consumer** — see `references/structure.md`.
4. **Composition over a base-component inheritance chain.**
5. **Templates stay styling-only, pages stay logic-only** — see `references/presentation.md`.
6. **A shared module earns its place on the third consumer** — except atoms/molecules, which start in `shared/ui`. See `references/presentation.md`.
7. **Keep async boundaries at the container** — see `references/state.md`.

## Troubleshooting

### A circular import appears right after adding a barrel file
**Fix**: import the concrete file, not the barrel, for in-feature/in-layer imports.

### A domain unit test suddenly needs a DI container or component harness to run
**Fix**: depend on the port interface, inject a hand-written fake.

### A "shared" folder has become a dumping ground nothing can be removed from
**Fix**: find each file's actual consumers; anything with exactly one moves back into that feature.

### A change to one API response breaks components in unrelated features
**Fix**: add the mapper at the infrastructure edge, type the use case's return as the entity.

See `references/antipatterns.md` for the full antipattern → replacement table.

## Constraints and Warnings

- **The dependency direction never inverts.**
- **No framework import inside domain or application.**
- **No DTO type used as a component prop or use-case return type.**
- **No business rule expressed inside a template or JSX/markup conditional.**
- **No deep import into another feature's internals** — go through its public entry.

## References

- **[references/layers.md](references/layers.md)** — open before scaffolding a feature or arguing where a file belongs.
- **[references/structure.md](references/structure.md)** — open before creating folders or wiring lint enforcement.
- **[references/presentation.md](references/presentation.md)** — open before decomposing or splitting a UI component.
- **[references/state.md](references/state.md)** — open before adding a store or deciding where a piece of state lives.
- **[references/clean-code.md](references/clean-code.md)** — open before writing or reviewing the code inside any layer.
- **[references/antipatterns.md](references/antipatterns.md)** — the antipattern → replacement table used in review/audit mode.
