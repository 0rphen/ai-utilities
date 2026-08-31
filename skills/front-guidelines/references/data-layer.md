# Data layer

Full code for every file mentioned here: `references/example-orders.md`.

## Port vs implementation

A port is an interface owned by `domain/`, expressed in domain vocabulary —
no HTTP verb, status code, SQL, or cache-TTL concept:

```ts
// domain/order.repository.port.ts
export interface OrderRepository {
  findById(id: string): Promise<Order>;
}
```

The implementation lives in `data/` and is the only thing allowed to know
transport/cache details.

## Repository is the only orchestrator

`ui/` and `domain/` never call a datasource directly — only the repository
implementation does, in every tier. This keeps remote/local swappable (e.g.
add a local cache later) without touching any caller.

In a `small`-tier feature (see `references/tiers.md`), the repository
implementation may make the transport call itself instead of delegating to a
separate `*.remote.datasource.ts` file — there's no second source to swap in
yet. The port and the mapper don't change, and `ui/` never notices either
way; see "Escalation path" below.

## Datasource port and swappable implementations

A datasource is a port too, scoped narrower than the repository. `remote`
talks to the network (HTTP/GraphQL/WebSocket — transport is an
implementation detail even within `data/`); `local` talks to a
cache/localStorage/IndexedDB. Both return the same DTO shape so the
repository can treat them interchangeably:

```ts
// data/order.datasource.port.ts
export interface OrderRemoteDatasource { fetchById(id: string): Promise<OrderDto>; }
export interface OrderLocalDatasource { get(id: string): Promise<OrderDto | null>; }
```

## Mapper is mandatory, both directions

No DTO type is ever imported by anything in `domain/` or `ui/`. If a
component needs a field that only exists on the DTO, that's a signal the
entity is incomplete — fix the entity and the mapper, don't leak the DTO.

## Error translation

Transport errors (HTTP 404, network timeout, GraphQL error payload) are
caught inside `data/` and translated into domain-meaningful errors (e.g.
`OrderNotFoundError`) before they reach `ui/`. A smart component or facade
catches and reacts to those, instead of branching on HTTP status codes.

## Where cache/query libraries live — and where they don't

TanStack Query, SWR, RTK Query, Angular's `resource()`/`httpResource()`,
Apollo's cache — all of these are **runtime/presentation concerns**. They
wrap a call to the repository; they do not replace it, and they never appear
inside `data/`. Reasoning: `data/` stays a plain, framework-free,
unit-testable module whose only job is "get me the entity." Swapping one
cache library for another, or migrating UI framework, never touches
`data/` or `domain/`.

## Escalation path

Growing `data/` from `small` to `medium`/`large` is additive, not a rewrite:

**Phase 1 (`small`, default starting point).** Port + one repository
implementation that makes the transport call inline + mapper. No datasource
files, no local cache.

```
domain/order.repository.port.ts
data/order.dto.ts
data/order.mapper.ts
data/order.repository.ts        # calls the network itself
```

**Phase 2 (`medium`/`large`, on trigger — a second data source, or caching/
offline logic appearing).** Extract the transport call behind
`order.remote.datasource.ts` (implementing a new `order.datasource.port.ts`),
optionally add `order.local.datasource.ts`, and have the repository
orchestrate both.

```
domain/order.repository.port.ts
data/order.dto.ts
data/order.mapper.ts
data/order.datasource.port.ts
data/order.remote.datasource.ts
data/order.local.datasource.ts   # added when caching/offline is needed
data/order.repository.ts         # now orchestrates remote + local
```

The repository's public shape (`OrderRepository`, from `domain/`) doesn't
change between phases, and neither does anything in `ui/` — that's the entire
point of the port living in `domain/` rather than being inferred from
whatever `data/` happens to do.

## Cache, TTL, and offline

Two different things get called "caching" here — keep them apart:

- **A cache/query library's own request cache** (TanStack Query's cache,
  `httpResource()`'s internal state, etc.) is a `ui/` concern — see "Where
  cache/query libraries live" above. It never enters `data/`.
- **A deliberate TTL/offline-fallback policy** (serve stale-but-present data
  when the network is down, treat cached data as expired after N minutes) is
  business-relevant orchestration logic, and it belongs in the repository
  implementation, coordinating a `remote` and a `local` datasource
  (`medium`/`large` tier — see the escalation path above). The repository
  decides *which* datasource answers a given call; `ui/` just awaits the
  repository method and never knows a cache was consulted.
