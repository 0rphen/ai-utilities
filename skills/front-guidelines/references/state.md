# State management

Where state lives and how it's shaped, independent of framework. See
`references/tiers.md` for the tier definitions this ladder is keyed to.

## The ladder

Climb only on a concrete trigger — never pre-emptively:

1. **Component-local** (`small` default, always the starting point). State
   used by exactly one component, gone when it unmounts. Plain
   the framework's local reactive-state primitive, no ceremony.
2. **Feature store** (`medium` and up, on trigger). Promote to
   `features/<x>/ui/<x>.store.*` when two or more components in the same
   feature need the same piece of state, or the same derived value would
   otherwise be recomputed and drift in more than one place.
3. **App-wide store** (`large`, or `medium` for genuinely global concerns).
   Lives in `core/` — session, theme, feature flags: state with exactly one
   instance for the app's lifetime, per the `core`/`shared` criterion in
   `references/structure.md`.
4. **Store library with devtools** (`large` only). Bring in a dedicated state
   library once app-wide state has real interdependencies across many
   features and the team needs time-travel/devtools debugging. Don't reach
   for this to manage a single feature's state — level 2 already covers that.

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
- **The store is obtained by DI/import/context, never passed as a prop.** A
  dumb component never receives a store (rule 8 in `SKILL.md`) — a smart
  component or facade reads it and passes plain data down.
- **Effects are for outgoing side-effects only** (persistence, logging,
  syncing to another system) — an effect must never write back to the state
  it depends on; that's a feedback loop, not a side-effect.

## A store is not a repository

A store calls the repository (rule 5 in `SKILL.md`) for anything that needs
to leave the process — it never calls a datasource directly, and it never
holds a DTO. What it holds is entities (or UI-shaped projections of them) —
the same mapper boundary from `references/data-layer.md` applies.

## Where a store lives

`ui/` (feature-scoped) or `core/` (app-wide) — never `domain/` (a store is a
runtime/presentation concern, not a business rule) and never `data/` (that
layer stays framework-free and has no notion of "current UI state").
