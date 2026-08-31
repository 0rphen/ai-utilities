# Project tier

The tier decides *how much* of `data/`, state, and routing gets deployed. It
never relaxes an invariant (domain port, mapper, smart/dumb split, feature
isolation) — those hold in every tier. See `SKILL.md` rule 1 for when to
resolve the tier at all.

## Declaration protocol

Read order: project `CLAUDE.md` → project `README.md` → ask.

The declaration lives in a fenced block in `CLAUDE.md`:

```markdown
<!-- front-guidelines -->
## Front guidelines
- project-tier: medium
<!-- /front-guidelines -->
```

If neither file has it and the current task's output actually depends on the
answer (scaffolding a new feature, adding a data layer, a store, or routes),
ask once via `AskUserQuestion` with the three tier definitions below as
options, then write the block above into `CLAUDE.md` (append if the file
exists, create it otherwise). Don't ask again once the block exists — re-read
it instead.

## When *not* to ask

- Reviewing or editing a single existing file: infer the tier from what's
  already there (does the feature have a `local` datasource? a facade? a
  dedicated store?) rather than interrupting the task.
- Any task whose output doesn't branch on tier (e.g. renaming a variable,
  fixing a typo, adding an entity field).

## No-`AskUserQuestion` fallback

Running without the tool (a subagent that lacks it, non-interactive mode): do
not block. Infer from the repo — number of features under `features/`,
whether any feature already has a `local` datasource, a facade, or a
dedicated store. Without a clear signal, default to `medium` and say so
explicitly in the response. Do **not** write a `medium` guess to `CLAUDE.md`
— only a value the user actually confirmed gets persisted.

## Tier definitions

| | small | medium | large |
|---|---|---|---|
| Typical scope | MVP, prototype, single-purpose tool | established product, several features | multi-team product, many interdependent features |
| Data sources per feature | one remote API | one, sometimes two (remote + cache) | several, offline/sync scenarios common |
| Team | 1 person | a few | multiple, possibly across squads |

## Decision matrix

| | small | medium | large |
|---|---|---|---|
| domain port + mapper + smart/dumb + `index.ts` | yes | yes | yes |
| datasource in its own file | no* — repository makes the call itself | yes | yes |
| local datasource / cache-TTL / offline | no* | optional | yes |
| facade | no* | with 2+ consumers | yes |
| pure domain service | optional | optional | yes |
| lazy per-feature routes | optional | yes | yes |
| state | component-local | feature store | shared/app-wide store |

`*` = only once the actual need appears — this is not a permanent exemption,
it's "not yet."

## What never changes

Regardless of tier: `domain/` port, mandatory mapper, hard smart/dumb split,
feature isolation via `index.ts` (+ `<feature>.routes.ts`, rule 10), the
`core`/`shared` criterion. `ui/` never imports a datasource and never sees a
DTO, in any tier — see `references/data-layer.md`'s escalation path for how
`small` still respects this.
