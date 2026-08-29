# Presentation

Full code for every component mentioned here: `references/example-orders.md`.

## The smart/dumb contract, precisely

A **smart** component:
- May call a facade, a repository, or a hook/composable wrapping either.
- Owns or reads server state, loading state, error state, empty state.
- May read router params, global/app state, or context.
- Passes derived data down to dumb components as props, and handles the
  events dumb components emit.

A **dumb** component:
- Receives everything through props/inputs. No other input source.
- Communicates only through emitted events/callbacks/outputs. No other output.
- **Never** imports anything from a feature's `data/` folder, directly or
  transitively.
- **Never** calls a repository, a datasource, or a facade.
- **Never** reads a router, global store, context bound to app state, or
  performs a fetch of any kind.
- Renders loading/error/empty only if explicitly told to via a prop
  (`isLoading: boolean`) — it does not decide *when* those states occur, only
  *how* they render if instructed.

This is a hard rule, not a style preference: a dumb component that imports
`data/` is a correctness violation of this skill, not a taste issue. It is
also what makes a dumb component trivially testable and reusable across
features.

## Where loading/error/empty are owned

Always in the smart component or the facade — never in the dumb component's
own logic. The dumb component may *render* a passed-in loading/error/empty
flag, but does not decide when that flag is true.

## When a facade is necessary — and when it isn't

Add a facade (`orders.facade.ts`) only when:
- Two or more smart components in the same feature need to share the same
  piece of state (e.g. a selected order, a filter), or
- The orchestration between repository calls and derived state is non-trivial
  enough that duplicating it across smart components would drift.

Skip the facade when a single smart component is the only consumer — it can
call the repository (wrapped in the framework's data-fetching primitive)
directly. Introducing a facade for a single consumer is indirection without
payoff.

A facade lives in `ui/`, not `domain/` or `data/` — it is a consumer of the
repository, exposing UI-shaped state (loading/error/data), not a producer of
domain contracts.

## Server state vs UI state

- Server state (anything that mirrors data from `data/`) is fetched and
  cached in `ui/`, via whatever cache/query primitive the framework/project
  uses (see `data-layer.md`'s "cache library" rule).
- UI-only state (a toggled filter, an open modal, a selected row not yet
  persisted anywhere) is local component or facade state — it has no
  business being modeled in `domain/` or persisted through a repository
  unless the product genuinely needs it to survive a reload.
