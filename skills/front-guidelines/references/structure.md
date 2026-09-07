# Structure

## Directory tree

```
src/
  features/
    orders/                    # one folder per business domain, not per tech layer
      domain/
        order.model.ts               # entity/model — data shape only
        order-repository.ts          # abstract class or interface (port)
      structure/
        order.dto.ts
        order.mapper.ts
        order-http-repository.ts     # implements/extends order-repository.ts
      store/
        orders.store.ts
      components/
        order-card.dumb.ts
      pages/
        order-list.page.ts           # the only thing orders.routes.ts mounts
      orders.facade.ts                # mandatory, the only surface pages/ calls
      orders.routes.ts
      index.ts                        # the ONLY public surface of this feature
    checkout/
      domain/ structure/ store/ components/ pages/ index.ts   # same shape, isolated
  core/                                  # runtime singletons, no business logic
    http/
      http-client.ts
      auth.interceptor.ts
    auth/
      auth.service.ts
    config/
      app-config.ts
    router/
      app-router.ts
    error/
      global-error-handler.ts
    store/
      session.store.ts               # app-wide store, one instance for the app
  shared/                                # general-purpose, visual, reusable, no domain knowledge
    button.dumb.ts
    input.dumb.ts
    card.dumb.ts
    header.dumb.ts
    footer.dumb.ts
    nav.dumb.ts
  utils/                                 # general-purpose pure functions — no UI, no state, no I/O
    format-date.ts
    pagination.ts
  styles/                                # only if the project's CSS policy centralizes
    tokens/
    blocks/
  app.routes.ts                          # composes and lazy-loads each feature's routes
```

Feature internals (`domain/`, `structure/`, `store/`, `components/`, `pages/`)
are an implementation detail. The only thing another feature — or the app
shell — may import is `index.ts` (plus `<feature>.routes.ts`, lazily).

## Mandatory naming fallback

Use the host framework's own convention first (rule 13 in `SKILL.md`). Apply
this table only when no such convention exists:

| Concern | Suffix |
|---|---|
| Domain entity/model | `*.model.ts` |
| Repository port (interface/abstract class) | `*-repository.ts` (in `domain/`) |
| Repository implementation | `*-http-repository.ts` (or transport-specific) |
| API response shape | `*.dto.ts` |
| DTO ↔ model mapper | `*.mapper.ts` |
| Feature or app-wide store | `*.store.ts` |
| Stateful orchestrator, mandatory per feature | `*.facade.ts` |
| Dumb, presentational component | `*.dumb.ts` (+ framework extension) |
| Smart, route-bound component | `*.page.ts` (+ framework extension) |
| Feature route definitions | `*.routes.ts` |

## `index.ts` barrel rule

Export only what other features or the app shell are allowed to consume —
typically: the model type(s), and any page component meant to be mounted
from outside the feature's own routes. Never re-export `structure/`,
`store/`, `domain/` ports, or the facade through the barrel; those exist only
for the feature's own `pages/`/`components/` to consume.

```ts
// features/orders/index.ts
export type { Order } from './domain/order.model';
export { OrderListPage } from './pages/order-list.page';
```

A feature's `<feature>.routes.ts` is the second public entry point, alongside
`index.ts` — it lives at the feature's root, outside
`domain|structure|store|components|pages`, so the feature-isolation grep
below already permits it without any exception. The app shell imports it
dynamically (lazy) rather than through a barrel — a barrel re-export would
defeat code-splitting.

## Feature isolation, mechanically

Forbidden: any import matching
`features/<X>/(domain|structure|store|components|pages)/` from a file
outside `features/<X>/`. Allowed: `features/<X>/index` from anywhere.
Auditable with a per-feature grep for that pattern excluding the feature's
own folder — a match means a deep import crossed a feature boundary.

## `core/` vs `shared/` vs `utils/` — worked examples

Ask, in order: "does this have exactly one instance while the app runs?" →
`core/`. Otherwise, "does it render UI?" → `shared/`. Otherwise → `utils/`.

| Item | Instance/shape | Folder |
|---|---|---|
| HTTP client + auth interceptor | One, shared app-wide | `core/` |
| Login/auth service, session state | One | `core/` |
| Router instance | One | `core/` |
| Global error handler / toast dispatcher | One | `core/` |
| `<Button>` / `<Input>` / `<Card>` dumb component | Many, stateless, visual | `shared/` |
| `<Header>` / `<Footer>` / `<Nav>` dumb component | Many, stateless, visual | `shared/` |
| `formatDate()` function | Many callers, no UI | `utils/` |
| `paginate()` / `groupBy()` function | Many callers, no UI | `utils/` |
| Pagination type/interface | N/A, pure type | `shared/` (co-located with what consumes it) |

If an item holds domain-specific meaning (an `Order`-shaped anything), it does
not belong in any of the three — it belongs inside the owning feature.
