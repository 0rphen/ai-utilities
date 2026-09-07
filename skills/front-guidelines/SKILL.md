---
name: front-guidelines
description: Feature-first (screaming) frontend architecture with a domain/structure data layer (declarations vs. implementations), a mandatory facade, hard smart/dumb component split, centralized styling, and SOLID/DRY principles. Use when creating a new frontend project, scaffolding a feature, or editing/reviewing existing frontend code for structure, data-access, state, routing, or component-boundary decisions.
allowed-tools: Read, Write, Edit, Glob, Grep, AskUserQuestion
---

# Front Guidelines

Framework-agnostic rules: folder tree screams the business domain, not the
tech stack. One canonical structure, no scaling tiers: feature-first
organization, a `domain`/`structure` data layer split (declarations vs.
implementations), a mandatory facade, hard smart/dumb split, centralized
token-driven styling, SOLID and DRY applied throughout. All generated code
(identifiers, file names, comments) is English, regardless of chat language.

## Instructions

1. **Name top folders by business domain, not tech type.** `features/orders/`,
   `features/checkout/` — never global `services/`, `models/`, `components/`
   buckets. See `references/structure.md`.

2. **Each feature = `domain/`, `structure/`, `store/`, `components/`,
   `pages/`, a facade file, a routes file.** One-way dependency
   `pages/components → facade → store/structure → domain`; `domain` imports
   nothing external.

3. **`domain/` declares, it never implements.** Entities/models, repository
   ports expressed as interfaces or abstract classes (domain vocabulary, no
   HTTP/SQL), and type-only contracts. No I/O, no framework import, nothing
   executable beyond a pure method on a model. See `references/data-layer.md`.

4. **`structure/` implements what `domain/` declares, flat.** One repository
   implementation per port, its mapper, and any transport/cache code it
   needs — no `datasource/` sub-layer, no remote/local port pair. The
   repository is the only orchestrator; nothing else in the feature calls a
   transport client directly. `structure/` never imports a UI/cache library.
   See `references/data-layer.md`.

5. **Mapper is mandatory** at the `structure`/`domain` boundary — DTOs never
   cross into `domain/`, `store/`, `components/`, or `pages/`.

6. **`components/`**: dumb, feature-scoped components. **`pages/`**: smart,
   route-bound components — the only things a route ever mounts.

7. **Hard smart/dumb boundary.** Dumb (`components/`) = props/inputs in,
   events/outputs out — nothing else. Dumb never imports `structure/`, never
   calls the facade/store/repository, never reads router/global state, never
   fetches, and never receives a store as a prop. Loading/error/empty are
   owned by `pages/`/the facade. See `references/presentation.md`.

8. **The facade is mandatory, and is the only surface `pages/` may call.**
   `pages/` never imports `store/` or `structure/` directly — the facade
   orchestrates the store and the repository and exposes UI-shaped state.
   See `references/presentation.md`.

9. **State escalates only on a concrete trigger.** Component-local
   `signal`/`state` first; promote to the feature's `store/` on 2+ consumers,
   duplicated derived state, or state that must survive navigation; promote
   to an app-wide store in `core/` only for genuinely global concerns —
   never pre-emptively. A store exposes private mutable state behind a
   readonly API, computes derived values itself, and is only ever mutated
   through named methods. See `references/state.md`.

10. **Hard feature isolation.** No file outside `features/<X>/` imports
    `features/<X>/(domain|structure|store|components|pages)/...` directly —
    only `features/<X>/index.ts` and `features/<X>/<X>.routes.ts`.
    Cross-feature reuse goes through `shared/`. See `references/structure.md`.

11. **Routing is per-feature, lazy-loaded.** Each feature declares its own
    `<feature>.routes.ts`; the app shell only composes and lazy-loads them,
    never defines feature-internal routes itself. See `references/structure.md`.

12. **`core/` vs `shared/` vs `utils/`: three criteria, not two.** Runtime
    singleton (HTTP client, interceptors, auth/login, global error handler,
    app-wide store) → `core/`. General-purpose *visual* building block
    reused across features, no domain knowledge (buttons, inputs, cards,
    headers, footers, navs) → `shared/`. General-purpose *pure function*, no
    UI, no state, no I/O → `utils/`. None of the three holds domain logic.
    See `references/structure.md`.

13. **Naming: follow the host framework's convention first.** Only when none
    exists, apply the mandatory fallback table in `references/structure.md`.

14. **Styling: delegate first.** Look for a project/user CSS guide; if one
    exists, follow it exactly. Otherwise, one-line fallback: consume
    centralized design tokens, no hardcoded values in component styles.

15. **Apply SOLID and DRY throughout.** One responsibility per file (SRP);
    depend on the `domain/` port, never the concrete `structure/`
    implementation (DIP); extend by adding a new port implementation, not by
    branching inside an existing one (OCP); never duplicate a mapping,
    validation, or formatting rule across features — lift it into `domain/`
    (business rule) or `shared/`/`utils/` (generic) instead (DRY). See
    `references/antipatterns.md`.

16. **All generated code is in English** — names, files, comments — even in a
    non-English conversation.

## Constraints

- No `application/`/use-case layer by default (rule 2) unless explicitly asked.
- No CSS methodology chapter here — only the delegation + fallback (rule 14).
- No executable scaffolding script — `references/example-orders.md` is a
  template to adapt, kept framework-agnostic.
- Testing conventions: out of scope.
- `domain/` never holds an implementation, and `structure/` never holds a
  declaration meant to be imported as the contract — the facade, the
  smart/dumb split, the mapper, and feature isolation apply unconditionally,
  with no tier or scale exception.

## References

- `references/structure.md` — tree, naming fallback table, barrel rule, `core`/`shared`/`utils` examples.
- `references/data-layer.md` — `domain`/`structure` contract, mapper, error translation, cache/query placement.
- `references/presentation.md` — smart/dumb detail, mandatory facade contract, state placement.
- `references/state.md` — state ladder, store encapsulation contract.
- `references/example-orders.md` — canonical end-to-end feature template.
- `references/antipatterns.md` — table of anti-patterns and fixes. Open when
  reviewing existing code, not needed when writing new code from scratch.
