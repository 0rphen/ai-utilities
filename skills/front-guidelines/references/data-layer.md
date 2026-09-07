# Data layer

Full code for every file mentioned here: `references/example-orders.md`.

## `domain/` declares, `structure/` implements

`domain/` holds only declarations: the model/entity shape, and a repository
port expressed as an interface or abstract class, in domain vocabulary — no
HTTP verb, status code, SQL, or cache-TTL concept:

```ts
// domain/order.model.ts
export interface Order {
  id: string;
  customerName: string;
  total: number;
}

// domain/order-repository.ts
export abstract class OrderRepository {
  abstract findById(id: string): Promise<Order>;
  abstract listByCustomer(customerId: string): Promise<Order[]>;
}
```

An abstract class works as well as an interface for a port — some
frameworks use it directly as a DI token. Either way, `domain/` contains no
method body that does real work: the only exception is a pure function/method
with no I/O and no framework import (e.g. a total-with-discount calculation
on the model itself). If a "pure business service" needs its own file, put
it in `structure/` — `domain/` is declarations only in this architecture.

The implementation lives in `structure/`, flat — no `datasource/`
sub-layer, no separate remote/local port pair:

```ts
// structure/order-http-repository.ts
export class OrderHttpRepository extends OrderRepository {
  async findById(id: string): Promise<Order> { /* ... */ }
  async listByCustomer(customerId: string): Promise<Order[]> { /* ... */ }
}
```

## Repository is the only orchestrator

`store/`, the facade, and `pages/`/`components/` never make a transport call
directly — only the repository implementation in `structure/` does. If a
feature later needs a second source (a local cache, an offline fallback),
that orchestration is added inside the same repository implementation; the
port in `domain/` and everything above it stay unchanged.

## Mapper is mandatory, both directions

No DTO type is ever imported by anything in `domain/`, `store/`,
`components/`, or `pages/`. `structure/` owns the DTO shape and the mapper
that converts it to/from the model declared in `domain/`. If a component
needs a field that only exists on the DTO, that's a signal the model is
incomplete — fix the model and the mapper, don't leak the DTO.

```ts
// structure/order.mapper.ts
import type { Order } from '../domain/order.model';
import type { OrderDto } from './order.dto';

export const OrderMapper = {
  toModel(dto: OrderDto): Order {
    return { id: dto.id, customerName: dto.customer_name, total: Number(dto.total_amount) };
  },
};
```

## Error translation

Transport errors (HTTP 404, network timeout, GraphQL error payload) are
caught inside `structure/` and translated into domain-meaningful errors
(e.g. `OrderNotFoundError`) before they reach the facade. The facade catches
and reacts to those, instead of branching on HTTP status codes.

## Where cache/query libraries live — and where they don't

TanStack Query, SWR, RTK Query, Angular's `resource()`/`httpResource()`,
Apollo's cache — all of these are **runtime/presentation concerns**. They
wrap a call to the facade (which itself calls the repository); they never
appear inside `structure/` or `domain/`. Reasoning: the data layer stays a
plain, framework-free, unit-testable module whose only job is "get me the
model." Swapping one cache library for another, or migrating UI framework,
never touches `structure/` or `domain/`.

A deliberate TTL/offline-fallback policy (serve stale-but-present data when
the network is down, treat cached data as expired after N minutes) is
business-relevant orchestration logic, and it belongs in the repository
implementation inside `structure/` — never in the facade, the store, or
`pages/`.
