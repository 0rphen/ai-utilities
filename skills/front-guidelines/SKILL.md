---
name: front-guidelines
description: Feature-first (screaming) frontend architecture with repository/datasource data layer, hard smart/dumb component split, tiered scaling (small/medium/large), and centralized styling. Use when creating a new frontend project, scaffolding a feature, or editing/reviewing existing frontend code for structure, data-access, state, routing, or component-boundary decisions.
allowed-tools: Read, Write, Edit, Glob, Grep, AskUserQuestion
---

# Front Guidelines

Framework-agnostic rules: folder tree screams the business domain, not the
tech stack. Five pillars: a declared project tier, feature-first structure,
repository/datasource data layer, hard smart/dumb split, centralized
token-driven styling. All generated code (identifiers, file names, comments)
is English, regardless of chat language.

## Instructions

1. **Determine the project tier before scaffolding or reviewing data/state/
   routing.** Read the `front-guidelines` block in the project's `CLAUDE.md`;
   if absent, check `README.md`; if still absent and the decision actually
   changes the output, ask via `AskUserQuestion` and persist the answer to
   `CLAUDE.md`. Skip asking for a read-only review of an existing file. See
   `references/tiers.md` for the protocol, the definitions, and the
   no-`AskUserQuestion` fallback.

2. **Name top folders by business domain, not tech type.**
   `features/orders/`, `features/checkout/` — never global `services/`,
   `models/`, `components/` buckets. See `references/structure.md`.

3. **Each feature = 3 layers: `domain/`, `data/`, `ui/`.** One-way dependency
   `ui → data → domain`; `domain` imports nothing external. No `application/`
   layer — logic needing it goes in a pure `domain/` service (rule 4).

4. **`domain/`**: entities, repository ports (domain-vocabulary interfaces,
   no HTTP/SQL), pure business services (`order-pricing.service.ts`, no I/O,
   unit-testable alone). See `references/data-layer.md`.

5. **`data/`**: repository implementation, mapper, and — from `medium` tier
   up — separate datasources (`remote`/`local`, interchangeable via a shared
   port). Repository is the only orchestrator — nothing else calls a
   datasource directly, in any tier. `data/` never imports a UI/cache
   library. See `references/data-layer.md` for the tiered shape and the
   escalation path.

6. **Mapper is mandatory** at the data/domain boundary — DTOs never cross
   into `domain/` or `ui/`, in any tier.

7. **`ui/`**: smart components, dumb components, optional facade (rule 8).

8. **Hard smart/dumb boundary.** Dumb = props/inputs in, events/outputs out —
   nothing else. Dumb never imports `data/`, never calls a repository/
   datasource/facade, never reads router/global state, never fetches, and
   never receives a store as a prop. Loading/error/empty are owned by
   smart/facade. See `references/presentation.md`.

9. **State follows the tier ladder.** Component-local `signal`/`state` first;
   escalate to a feature store, then an app-wide store, only on a concrete
   trigger (2+ consumers, duplicated derived state, or state that must
   survive navigation) — never pre-emptively. A store exposes private
   mutable state behind a readonly API, computes derived values itself, and
   is only ever mutated through named methods. See `references/state.md`.

10. **Hard feature isolation.** No file outside `features/<X>/` imports
    `features/<X>/(domain|data|ui)/...` directly — only `features/<X>/
    index.ts` and `features/<X>/<X>.routes.ts`. Cross-feature reuse goes
    through `shared/`. See `references/structure.md`.

11. **Routing is per-feature, lazy-loaded from `medium` tier up.** Each
    feature declares its own `<feature>.routes.ts`; the app shell only
    composes and lazy-loads them, never defines feature-internal routes
    itself. See `references/structure.md`.

12. **`core/` vs `shared/`: one criterion — runtime singleton vs. stateless
    reusable.** Single instance at runtime (HTTP client, router, global error
    handler, app-wide store) → `core/`. Reused many times, no state (dumb
    primitives, utils, generic hooks/types) → `shared/`. Neither holds
    domain logic.

13. **Naming: follow the host framework's convention first.** Only when none
    exists, apply the mandatory fallback table in `references/structure.md`.

14. **Styling: delegate first.** Look for a project/user CSS guide; if one
    exists, follow it exactly. Otherwise, one-line fallback: consume
    centralized design tokens, no hardcoded values in component styles.

15. **All generated code is in English** — names, files, comments — even in a
    non-English conversation.

## Constraints

- No `application/`/use-case layer by default (rule 3) unless explicitly asked.
- No CSS methodology chapter here — only the delegation + fallback (rule 14).
- No executable scaffolding script — `references/example-orders.md` is a
  template to adapt, kept framework-agnostic.
- Testing conventions: out of scope.
- Tier affects *how much* of `data/`, state, and routing is deployed — it
  never relaxes the domain port, the mapper, the smart/dumb split, or feature
  isolation. `ui/` never calls a datasource or sees a DTO, in any tier.

## References

- `references/tiers.md` — tier declaration protocol, definitions, decision matrix, fallback.
- `references/structure.md` — tree, naming fallback table, barrel rule, `core`/`shared` examples.
- `references/data-layer.md` — port/impl contract, tiered `data/` shape, escalation path, cache/TTL/offline.
- `references/presentation.md` — smart/dumb detail, facade necessity test, state placement.
- `references/state.md` — state ladder by tier, store encapsulation contract.
- `references/example-orders.md` — canonical end-to-end feature template.
- `references/antipatterns.md` — table of anti-patterns and fixes. Open when
  reviewing existing code, not needed when writing new code from scratch.
