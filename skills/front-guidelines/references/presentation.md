# Presentation

Full code for every component mentioned here: `references/example-orders.md`.

## The smart/dumb contract, precisely

A **smart** component (`pages/`):
- May call the feature's facade — nothing else. Never imports `structure/`
  or `store/` directly (rule 8 in `SKILL.md`).
- Owns or reads server state, loading state, error state, empty state, as
  exposed by the facade.
- May read router params, global/app state, or context.
- Passes derived data down to dumb components as props, and handles the
  events dumb components emit.

A **dumb** component (`components/`, or a general-purpose one in `shared/`):
- Receives everything through props/inputs. No other input source.
- Communicates only through emitted events/callbacks/outputs. No other output.
- **Never** imports anything from a feature's `structure/` or `store/`
  folder, directly or transitively.
- **Never** calls a repository, the facade, or the store.
- **Never** receives a store as a prop/input — a store is DI'd/imported by
  the facade, which passes plain data down (see `references/state.md`).
- **Never** reads a router, global store, context bound to app state, or
  performs a fetch of any kind.
- Renders loading/error/empty only if explicitly told to via a prop
  (`isLoading: boolean`) — it does not decide *when* those states occur, only
  *how* they render if instructed.

This is a hard rule, not a style preference: a dumb component that imports
`structure/` is a correctness violation of this skill, not a taste issue. It
is also what makes a dumb component trivially testable and reusable across
features.

## Where loading/error/empty are owned

Always in the facade, surfaced to `pages/` — never decided inside a dumb
component's own logic. The dumb component may *render* a passed-in
loading/error/empty flag, but does not decide when that flag is true.

## The facade contract — mandatory, not optional

Every feature has exactly one facade (`<feature>.facade.ts`), and it is the
only thing `pages/` may call:

- Orchestrates the store and the repository (via `domain/`'s port): calls
  repository methods, writes results into the store, exposes UI-shaped
  state (loading/error/data) that `pages/` reads.
- Is the single place that duplicated orchestration would otherwise drift
  across multiple `pages/` in the same feature — even a feature with one
  page keeps the facade, so adding a second consumer later never requires
  extracting one out of a page.
- Lives at the feature root, not inside `domain/` or `structure/` — it is a
  consumer of the repository and the store, exposing UI-shaped state, not a
  producer of domain contracts.

## Server state vs UI state

- Server state (anything that mirrors data from `structure/`) is fetched via
  the repository and surfaced through the facade, optionally via whatever
  cache/query primitive the framework/project uses (see `data-layer.md`'s
  "cache library" rule).
- UI-only state (a toggled filter, an open modal, a selected row not yet
  persisted anywhere) is local component or facade state — it has no
  business being modeled in `domain/` or persisted through a repository
  unless the product genuinely needs it to survive a reload.
- For how far that state should be lifted (component-local vs. a feature or
  app-wide store) and the encapsulation contract a store must follow, see
  `references/state.md`.
