# State management

Where state lives and how it's shaped, independent of framework.

## The ladder

Climb only on a concrete trigger — never pre-emptively:

1. **Component-local.** Always the starting point. State used by exactly one
   component (a `page` or a `component`), gone when it unmounts. Plain the
   framework's local reactive-state primitive, no ceremony.
2. **Feature store** (`features/<x>/store/<x>.store.*`). Promote on a
   concrete trigger: two or more components in the same feature need the
   same piece of state, or the same derived value would otherwise be
   recomputed and drift in more than one place, or the state must survive
   navigation within the feature.
3. **App-wide store** (`core/store/`). Only for genuinely global concerns —
   session, theme, feature flags: state with exactly one instance for the
   app's lifetime, per the `core`/`shared`/`utils` criterion in
   `references/structure.md`.
4. **Store library with devtools.** Bring in a dedicated state library only
   once app-wide state has real interdependencies across many features and
   the team needs time-travel/devtools debugging. Don't reach for this to
   manage a single feature's state — level 2 already covers that.

Triggers to escalate: a second consumer needs the same state; a derived value
is duplicated across components; the state must survive route navigation.
None of these being true yet means: stay at the current level.

## Store contract (any level ≥ 2)

- **Private mutable state, public readonly API.** Nothing outside the store
  writes to the underlying state directly.
- **Derived values are computed inside the store**, once, not recomputed in
  every render/template that reads them.
- **Mutation only through named methods** (`addItem`, `clear`, …) — never a
  raw setter exposed to callers. This keeps every state change traceable to
  one call site's worth of intent.
- **The store is obtained by DI/import/context, and only the facade reads
  it — never passed as a prop.** A dumb component never receives a store
  (rule 7 in `SKILL.md`); the facade reads it and passes plain data down to
  `pages/`.
- **Effects are for outgoing side-effects only** (persistence, logging,
  syncing to another system) — an effect must never write back to the state
  it depends on; that's a feedback loop, not a side-effect.

## A store is not a repository

A store never calls a transport client or a repository directly — it holds
state that the **facade** populates by calling the repository (rule 8 in
`SKILL.md`). It never holds a DTO. What it holds is models (or UI-shaped
projections of them) — the same mapper boundary from
`references/data-layer.md` applies.

## Where a store lives

`features/<x>/store/` (feature-scoped) or `core/store/` (app-wide) — never
`domain/` (a store is a runtime/presentation concern, not a business rule)
and never `structure/` (that layer stays framework-free and has no notion of
"current UI state").
