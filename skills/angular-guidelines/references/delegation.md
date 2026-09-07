# Delegation

This skill decides *which* pattern and *where* it lives. It never restates an
API — that's always a delegated skill's job.

## Version detection

Read order: `package.json` `@angular/core` version → Angular CLI/MCP tooling
(if available, e.g. `get_best_practices`) → existing project conventions.
Never assume a major when naming a primitive. Version changes which API is
current — it never changes an architectural rule in `SKILL.md`.

## Ownership map

| Question | Owner |
|---|---|
| Signals, `linkedSignal`, `resource`, `effect` semantics | `angular-developer` (`signals-overview.md`, `resource.md`, `effects.md`) |
| Forms (signal/reactive/template-driven) | `angular-developer` (`signal-forms.md`, `reactive-forms.md`, `template-driven-forms.md`) |
| DI mechanics, injection context, providers | `angular-developer` (`di-fundamentals.md`, `creating-services.md`, `injection-context.md`) |
| HTTP client, interceptors, `httpResource` | `angular-developer` (`http-client.md`) |
| Route definitions, guards, resolvers, SSR/hydration | `angular-developer` (routing reference set) |
| Testing (unit, harnesses, e2e) | `angular-developer` (`testing-fundamentals.md` and siblings) |
| CLI, schematics, migrations | `angular-developer` (`cli.md`) |
| New project bootstrap (`ng new`, flags) | `angular-new-app` |
| Feature-first structure, `domain/structure`, ports, facade | `front-guidelines` |
| CSS/SCSS authoring | `css-guidelines` |
| Which pattern, which boundary, which scope | this skill |

If a delegated skill isn't installed: apply the principle from `SKILL.md`,
state the assumption made, and don't invent API detail on its behalf.

## Precedence

`front-guidelines` is the source of truth for `domain/structure`, the mapper,
the mandatory facade, and feature isolation whenever it's installed — this
skill only binds those to Angular concepts (which files, which DI scope).

`angular-developer/references/naming-conventions.md` also prescribes a
`core/features/shared` structure and file naming. For **architecture**
(layering, folder boundaries), `front-guidelines` + this skill win. For
**file/class suffix conventions** matching the installed Angular version,
`angular-developer` wins.

## Standalone fallback

Without `front-guidelines` installed, use this minimal shape per feature:

```
features/<name>/
  domain/       # models, repository port (interface/abstract class) — no Angular import, no implementation
  structure/    # repository impl, mapper, HTTP client usage — flat
  store/        # feature state, private writable signal behind a readonly API
  components/   # dumb, feature-scoped presentational components
  pages/        # smart, route-bound components
  <name>.facade.ts   # mandatory, the only thing pages/ calls
```
