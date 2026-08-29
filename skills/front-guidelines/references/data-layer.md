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
implementation does. This keeps remote/local swappable (e.g. add a local
cache later) without touching any caller.

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
