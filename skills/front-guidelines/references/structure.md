# Structure

## Directory tree

```
src/
  features/
    orders/                    # one folder per business domain, not per tech layer
      domain/
        order.entity.ts
        order.repository.port.ts
        order-pricing.service.ts
      data/
        order.dto.ts
        order.mapper.ts
        order.remote.datasource.ts
        order.local.datasource.ts        # optional: cache/offline
        order.datasource.port.ts
        order.repository.ts
      ui/
        orders.facade.ts                 # optional, see presentation.md
        orders.store.ts                  # optional, feature-level, see state.md
        order-list.smart.ts
        order-card.dumb.ts
      orders.routes.ts                   # optional in small tier, see tiers.md
      index.ts                           # the ONLY public surface of this feature
    checkout/
      domain/ data/ ui/ index.ts         # same shape, isolated from orders/
  core/                                  # runtime singletons, no business logic
    http/
      http-client.ts
    config/
      app-config.ts
    router/
      app-router.ts
    error/
      global-error-handler.ts
  shared/                                # stateless, reusable, no business logic
    ui/
      button.dumb.ts
      spinner.dumb.ts
    utils/
      format-date.ts
    types/
      pagination.ts
  styles/                                # only if the project's CSS policy centralizes
    tokens/
    blocks/
  app.routes.ts                          # composes and lazy-loads each feature's routes
```

Feature internals (`domain/`, `data/`, `ui/`) are an implementation detail. The
only thing another feature — or the app shell — may import is `index.ts`.

## Mandatory naming fallback

Use the host framework's own convention first (rule 13 in `SKILL.md`). Apply
this table only when no such convention exists:

| Concern | Suffix |
|---|---|
| Domain entity | `*.entity.ts` |
| Repository port (interface) | `*.repository.port.ts` |
| Repository implementation | `*.repository.ts` |
| Datasource port (interface) | `*.datasource.port.ts` |
| Remote datasource implementation | `*.remote.datasource.ts` |
| Local datasource implementation | `*.local.datasource.ts` |
| API response shape | `*.dto.ts` |
| DTO ↔ entity mapper | `*.mapper.ts` |
| Pure business-logic service | `*.service.ts` |
| Stateful UI orchestrator | `*.facade.ts` |
| Feature or app-wide store | `*.store.ts` |
| Presentational component with logic | `*.smart.ts` (+ framework extension) |
| Presentational component without logic | `*.dumb.ts` (+ framework extension) |
| Feature route definitions | `*.routes.ts` |

## `index.ts` barrel rule

Export only what other features or the app shell are allowed to consume —
typically: the entity type(s), and the smart component(s) meant to be mounted
from outside. Never re-export `data/` contents, ports, or internal services
through the barrel; those exist only for the feature's own `ui/` layer to
consume.

```ts
// features/orders/index.ts
export type { Order } from './domain/order.entity';
export { OrderListSmart } from './ui/order-list.smart';
```

A feature's `<feature>.routes.ts` is the second public entry point, alongside
`index.ts` — it lives at the feature's root, outside `domain|data|ui`, so the
feature-isolation grep below already permits it without any exception. The
app shell imports it dynamically (lazy) rather than through a barrel — a
barrel re-export would defeat code-splitting.

## Feature isolation, mechanically

Forbidden: any import matching `features/<X>/(domain|data|ui)/` from a file
outside `features/<X>/`. Allowed: `features/<X>/index` from anywhere.
Auditable with a per-feature grep for that pattern excluding the feature's
own folder — a match means a deep import crossed a feature boundary.

## `core/` vs `shared/` — worked examples

Ask: "does this have exactly one instance while the app runs?"

| Item | Instance? | Folder |
|---|---|---|
| HTTP client wrapping fetch/axios | One, shared app-wide | `core/` |
| Router instance | One | `core/` |
| Global error handler / toast dispatcher | One | `core/` |
| App-wide auth/session state | One | `core/` |
| `<Button>` dumb component | Many, stateless | `shared/` |
| `formatDate()` utility | Many, stateless | `shared/` |
| Generic `usePagination()` hook | Many, no shared state between callers | `shared/` |
| Pagination type/interface | N/A, pure type | `shared/` |

If an item holds domain-specific meaning (an `Order`-shaped anything), it does
not belong in either — it belongs inside the owning feature.
