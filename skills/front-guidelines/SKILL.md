---
name: front-guidelines
description: Feature-first (screaming) frontend architecture with repository/datasource data layer, hard smart/dumb component split, and centralized styling. Use when creating a new frontend project, scaffolding a feature, or editing/reviewing existing frontend code for structure, data-access, or component-boundary decisions.
allowed-tools: Read, Write, Edit, Glob, Grep
---

# Front Guidelines

Framework-agnostic rules: folder tree screams the business domain, not the
tech stack. Four pillars: feature-first structure, repository/datasource data
layer, hard smart/dumb split, centralized token-driven styling. All generated
code (identifiers, file names, comments) is English, regardless of chat language.

## Instructions

1. **Name top folders by business domain, not tech type.**
   `features/orders/`, `features/checkout/` — never global `services/`,
   `models/`, `components/` buckets. See `references/structure.md`.

2. **Each feature = 3 layers: `domain/`, `data/`, `ui/`.** One-way dependency
   `ui → data → domain`; `domain` imports nothing external. No `application/`
   layer — logic needing it goes in a pure `domain/` service (rule 3).

3. **`domain/`**: entities, repository ports (domain-vocabulary interfaces,
   no HTTP/SQL), pure business services (`order-pricing.service.ts`, no I/O,
   unit-testable alone). See `references/data-layer.md`.

4. **`data/`**: datasources (`remote`/`local`, interchangeable via a shared
   port), mappers, repository implementation. Repository is the only
   orchestrator — nothing else calls a datasource directly. `data/` never
   imports a UI/cache library. See `references/data-layer.md`.

5. **Mapper is mandatory** at the data/domain boundary — DTOs never cross
   into `domain/` or `ui/`.

6. **`ui/`**: smart components, dumb components, optional facade (rule 7).

7. **Hard smart/dumb boundary.** Dumb = props/inputs in, events/outputs out —
   nothing else. Dumb never imports `data/`, never calls a repository/
   datasource/facade, never reads router/global state, never fetches.
   Loading/error/empty are owned by smart/facade. See `references/presentation.md`.

8. **Hard feature isolation.** No file outside `features/<X>/` imports
   `features/<X>/(domain|data|ui)/...` directly — only `features/<X>/index.ts`.
   Cross-feature reuse goes through `shared/`. See `references/structure.md`.

9. **`core/` vs `shared/`: one criterion — runtime singleton vs. stateless
   reusable.** Single instance at runtime (HTTP client, router, global error
   handler) → `core/`. Reused many times, no state (dumb primitives, utils,
   generic hooks/types) → `shared/`. Neither holds domain logic.

10. **Naming: follow the host framework's convention first.** Only when none
    exists, apply the mandatory fallback table in `references/structure.md`.

11. **Styling: delegate first.** Look for a project/user CSS guide; if one
    exists, follow it exactly. Otherwise, one-line fallback: consume
    centralized design tokens, no hardcoded values in component styles.

12. **All generated code is in English** — names, files, comments — even in a
    non-English conversation.

## Constraints

- No `application/`/use-case layer by default (rule 2) unless explicitly asked.
- No CSS methodology chapter here — only the delegation + fallback (rule 11).
- No executable scaffolding script — `references/example-orders.md` is a
  template to adapt, kept framework-agnostic.
- Testing conventions: out of scope.

## References

- `references/structure.md` — tree, naming fallback table, barrel rule, `core`/`shared` examples.
- `references/data-layer.md` — port/impl contract, datasource strategy, error translation, mapper.
- `references/presentation.md` — smart/dumb detail, facade necessity test, state placement.
- `references/example-orders.md` — canonical end-to-end feature template.
- `references/antipatterns.md` — table of anti-patterns and fixes. Open when
  reviewing existing code, not needed when writing new code from scratch.
