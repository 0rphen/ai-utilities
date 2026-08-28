# Frontend Architecture — Layers

Read this whenever a feature is being scaffolded, a file needs a home, or a proposed change would make a lower layer depend on a higher one.

## The Dependency Rule

Source code dependencies point in one direction only: **presentation → infrastructure → application → domain**, or straight from presentation into application. Nothing inside domain imports from application, infrastructure, or presentation. Nothing inside application imports from infrastructure or presentation. The inner layers know nothing about how they're consumed or how their data arrives — they just declare the shapes and the interfaces the outer layers must satisfy.

This is what makes each layer independently testable: domain and application run under a plain test runner with no DOM, no framework, no network. If a domain test needs to mock a component or spin up a router, a dependency pointed the wrong way.

## Domain

The business entities and the rules that must always hold for them. A domain type is a plain class or plain object with no decorator, no framework base class, no injected service:

```typescript
// order.entity.ts
export class Order {
  constructor(
    public readonly id: string,
    public readonly items: OrderLine[],
    public readonly status: OrderStatus,
  ) {}

  get total(): number {
    return this.items.reduce((sum, line) => sum + line.quantity * line.unitPrice, 0);
  }

  canBeCancelled(): boolean {
    return this.status === 'pending' || this.status === 'confirmed';
  }
}
```

Domain also owns the **repository and gateway interfaces** — the ports the rest of the app needs, defined in terms domain cares about, not in terms of HTTP or SQL:

```typescript
// order-repository.port.ts
export interface OrderRepository {
  findById(id: string): Promise<Order | null>;
  save(order: Order): Promise<void>;
}
```

Nothing in this file imports `fetch`, an ORM, or a UI framework. The interface is domain's demand; infrastructure supplies it.

## Application

Use cases: one class or function per user-facing operation, orchestrating domain objects and ports, with no knowledge of how it's invoked (an HTTP handler, a form submit, a CLI command — the use case doesn't know or care).

```typescript
// cancel-order.usecase.ts
export class CancelOrderUseCase {
  constructor(private readonly orders: OrderRepository) {}

  async execute(orderId: string): Promise<Order> {
    const order = await this.orders.findById(orderId);
    if (!order) throw new OrderNotFoundError(orderId);
    if (!order.canBeCancelled()) throw new OrderNotCancellableError(orderId);

    const cancelled = new Order(order.id, order.items, 'cancelled');
    await this.orders.save(cancelled);
    return cancelled;
  }
}
```

Note `canBeCancelled()` — the invariant check — lives on the entity, not the use case. The use case orchestrates; the entity protects its own rules. If the check only ever happens in one place and never needs to be reused or unit-tested in isolation, it's fine to inline it in the use case instead — don't manufacture an entity method for a rule that's used once.

## Infrastructure

Adapters that implement the ports domain declared, plus DTOs and the mappers that translate between wire shape and entity. This is the only layer allowed to know about HTTP, storage, or a third-party SDK.

```typescript
// order.dto.ts
export interface OrderDto {
  id: string;
  line_items: { sku: string; qty: number; unit_price_cents: number }[];
  status_code: 'PEND' | 'CONF' | 'CANC' | 'DONE';
}

// order.mapper.ts
const STATUS_MAP: Record<OrderDto['status_code'], OrderStatus> = {
  PEND: 'pending', CONF: 'confirmed', CANC: 'cancelled', DONE: 'fulfilled',
};

export function toOrderEntity(dto: OrderDto): Order {
  return new Order(
    dto.id,
    dto.line_items.map((li) => new OrderLine(li.sku, li.qty, li.unit_price_cents / 100)),
    STATUS_MAP[dto.status_code],
  );
}

// http-order-repository.adapter.ts
export class HttpOrderRepository implements OrderRepository {
  constructor(private readonly http: HttpClient) {}

  async findById(id: string): Promise<Order | null> {
    const dto = await this.http.get<OrderDto>(`/orders/${id}`);
    return dto ? toOrderEntity(dto) : null;
  }

  async save(order: Order): Promise<void> {
    await this.http.put(`/orders/${order.id}`, toOrderDto(order));
  }
}
```

`snake_case` field names, cents-as-integers, a status-code enum — none of that ever appears outside this file. Past the mapper, the rest of the app only ever sees `Order`.

## Presentation

Pages call use cases and hold the result as state; everything below a page renders what it's given. See `references/presentation.md` for how presentation itself decomposes — this layer is wide enough to need its own reference.

## A Trace Across All Four Layers

One operation, framework-neutral, showing the full inward-then-outward round trip:

```typescript
// composition root — the only place all four layers are wired together
const httpClient = new FetchHttpClient(baseUrl);
const orderRepository = new HttpOrderRepository(httpClient);   // infrastructure implements domain's port
const cancelOrder = new CancelOrderUseCase(orderRepository);   // application depends on domain only

// presentation — a page/container, framework of choice
async function onCancelClick(orderId: string) {
  try {
    const order = await cancelOrder.execute(orderId);  // crosses into application
    setOrder(order);                                   // renders the entity, never the DTO
  } catch (err) {
    if (err instanceof OrderNotCancellableError) setError('This order can no longer be cancelled.');
  }
}
```

The page never sees `OrderDto`, never calls `http.put` directly, and never duplicates `canBeCancelled()` — it asks the use case and renders the result or the domain error.

## Collapse Table

Not every project earns all four layers. Collapse rather than force a layer that has nothing to protect:

| Project shape | What to keep |
| --- | --- |
| Static/marketing site, no business rules | Presentation only — no domain, no use cases. |
| Simple CRUD admin panel, thin validation | Domain entities optional; a typed API client can stand in for repository + mapper if the DTO and the display shape are already identical. |
| One or two features with real invariants (pricing, eligibility, state machines) | Full four layers for those features; simpler features stay collapsed. Mixed depth in the same codebase is normal. |
| Product with shared business rules consumed by multiple frontends (web + mobile) or by a backend too | Full four layers, and consider domain/application as a framework-free package importable by every consumer. |

The signal to add a layer is a concrete pain: a rule duplicated in two places, a component that can't be unit-tested without a live server, a DTO change breaking three unrelated screens. Add the seam that removes that specific pain, not the whole stack preemptively.
