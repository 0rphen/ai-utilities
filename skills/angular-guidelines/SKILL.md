---
name: angular-guidelines
description: Angular-specific architecture and policy rules — where state lives, data-access boundaries, component boundaries, feature structure, and DI/routing/change-detection policy. Version-independent; defers API syntax, forms, and CLI to angular-developer. Use when creating or reviewing Angular application architecture, or making a state-placement, data-access, or component-boundary decision.
allowed-tools: Read, Write, Edit, Glob, Grep, AskUserQuestion
---

# Angular Guidelines

Angular's architecture and policy layer: which pattern, which boundary, which
scope — never which API call. Companion to `front-guidelines` (feature-first
structure, ports, tiers) and to `angular-developer`/`angular-new-app` (API
mechanics, project bootstrap). Version-independent: no primitive named here is
pinned to a major — check it against the project's installed Angular version.
All generated code (identifiers, files, comments) is English, regardless of
chat language.

## Instructions

1. **Detect the installed Angular version before naming any API.** Read
   `package.json`; use Angular CLI/MCP tooling when present; never assume a
   major. Version gates which primitive, never which architecture. See
   `references/delegation.md`.
2. **Delegate API mechanics — never restate them.** Signals/`resource`, forms,
   DI mechanics, routing APIs, SSR, a11y, animations, testing, CLI →
   `angular-developer`; project creation → `angular-new-app`. This skill
   decides which and where, not how to type it. See `references/delegation.md`.
3. **Structure comes from `front-guidelines`; this skill only binds it to
   Angular.** `domain/data/ui`, mapper, feature isolation, tier ladder take
   precedence over `angular-developer`'s `naming-conventions.md` for
   architecture. See `references/delegation.md`.
4. **Classify by responsibility, not by folder or feature name.** One feature
   may hold both infrastructure and application responsibilities — split by
   what the code does, not what it's named.
5. **State follows the ladder, simplest mechanism that fits the scope.**
   Component-local → feature store → app-wide store → store library,
   escalating only on a concrete trigger. See `references/state.md`.
6. **Encapsulate writable state, expose it read-only.** Private writable
   signal, public readonly accessor, derived state computed not duplicated,
   mutation only via named methods. See `references/state.md`.
7. **Signals for state, streams for stream-shaped flows.** Reach for RxJS on
   composition, cancellation, concurrency, buffering, polling, continuous
   events; convert at explicit boundaries; never expose an observable as the
   primary UI state interface. See `references/state.md`.
8. **Effects are for outgoing side effects only.** Never a state-sync
   mechanism; never write back to state the effect reads. See
   `references/state.md`.
9. **Data access stays behind the repository boundary.** No HTTP client, URL,
   DTO, or transport error reaches a component; map at the boundary;
   resource/query primitives live in `ui/` wrapping the repository, never in
   `data/`. See `references/boundaries.md`.
10. **Injectables have one coherent responsibility, scoped to the lifetime of
    the state they own.** No god services; no interface/token without a
    substitution or testability reason. See `references/boundaries.md`.
11. **Components coordinate or present.** Smart/dumb boundary; declarative
    templates with no business logic or derivation; explicit input/output
    contracts; no premature extraction; change-detection policy (zoneless
    where the installed version supports it, otherwise `OnPush`) declared once,
    project-wide. See `references/boundaries.md`.
12. **Routing is a composition/navigation boundary, and infrastructure stays
    out of application logic.** Per-feature route files composed by the shell;
    guards decide access, not business workflows; route/query params are
    explicit inputs, never duplicated into state; interceptors, auth
    mechanism, persistence, logging, and telemetry sit behind clear
    boundaries. See `references/boundaries.md`.

## Constraints

- No API tutorials, syntax, or CLI commands — `angular-developer`/
  `angular-new-app` own those; report a conflict rather than restating them.
- No version pinning. Any primitive named in `references/` is illustrative of
  current-era Angular and must be checked against the installed version.
- `front-guidelines` is the structure source of truth when installed; this
  skill's fallback in `references/delegation.md` applies only standalone.
- Precedence over `angular-developer`'s `naming-conventions.md` covers
  architecture (folders, layering) only — it still wins on file/class suffix
  conventions for the installed version.
- Testing conventions and CSS methodology: out of scope.

## References

- `references/delegation.md` — version detection, ownership map, precedence, standalone fallback tree.
- `references/state.md` — state ladder, store contract, signals vs RxJS, effect rules.
- `references/boundaries.md` — data layer, structure/DI binding, smart/dumb presentation rules.
- `references/antipatterns.md` — table of anti-patterns and fixes. Open when reviewing existing code, not needed when writing new code from scratch.
