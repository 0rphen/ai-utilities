# Canonical example: `orders`

Framework-agnostic TypeScript. This is the template to adapt — file names,
folder shape, and layer boundaries transfer as-is; syntax adapts to whatever
framework the project uses.

```
features/orders/
  domain/
    order.model.ts
    order-repository.ts
  structure/
    order.dto.ts
    order.mapper.ts
    order-http-repository.ts
  store/
    orders.store.ts
  components/
    order-card.dumb.ts
  pages/
    order-list.page.ts
  orders.facade.ts
  orders.routes.ts
  index.ts
```

### `domain/order.model.ts`

```ts
export interface Order {
  id: string;
  customerName: string;
  items: OrderItem[];
  total: number;
  placedAt: Date;
}

export interface OrderItem {
  sku: string;
  quantity: number;
  unitPrice: number;
}
```

### `domain/order-repository.ts`

```ts
import type { Order } from './order.model';

/** Port: declaration only. structure/ provides the implementation. */
export abstract class OrderRepository {
  abstract findById(id: string): Promise<Order>;
  abstract listByCustomer(customerId: string): Promise<Order[]>;
}
```

### `structure/order.dto.ts`

```ts
export interface OrderDto {
  id: string;
  customer_name: string;
  items: { sku: string; qty: number; unit_price: number }[];
  total_amount: string; // API returns a string
  placed_at: string;    // ISO date string
}
```

### `structure/order.mapper.ts`

```ts
import type { Order } from '../domain/order.model';
import type { OrderDto } from './order.dto';

export const OrderMapper = {
  toModel(dto: OrderDto): Order {
    return {
      id: dto.id,
      customerName: dto.customer_name,
      items: dto.items.map(i => ({ sku: i.sku, quantity: i.qty, unitPrice: i.unit_price })),
      total: Number(dto.total_amount),
      placedAt: new Date(dto.placed_at),
    };
  },
};
```

### `structure/order-http-repository.ts`

```ts
import { OrderRepository } from '../domain/order-repository';
import type { Order } from '../domain/order.model';
import type { OrderDto } from './order.dto';
import { OrderMapper } from './order.mapper';
import { httpClient } from '../../../core/http/http-client';
import { OrderNotFoundError, OrderFetchError } from '../domain/order.errors';

export class OrderHttpRepository extends OrderRepository {
  async findById(id: string): Promise<Order> {
    const res = await httpClient.get(`/orders/${id}`);
    if (res.status === 404) throw new OrderNotFoundError(id);
    if (!res.ok) throw new OrderFetchError(res.status);
    const dto: OrderDto = await res.json();
    return OrderMapper.toModel(dto);
  }

  async listByCustomer(customerId: string): Promise<Order[]> {
    const res = await httpClient.get(`/customers/${customerId}/orders`);
    if (!res.ok) throw new OrderFetchError(res.status);
    const dtos: OrderDto[] = await res.json();
    return dtos.map(OrderMapper.toModel);
  }
}
```

### `store/orders.store.ts`

```ts
import type { Order } from '../domain/order.model';

/** Private mutable state, public readonly API, mutation only via named methods. */
export class OrdersStore {
  #orders: Order[] = [];

  get orders(): readonly Order[] { return this.#orders; }

  setOrders(orders: Order[]) { this.#orders = orders; }
}
```

### `orders.facade.ts`

```ts
import { useState } from 'react'; // example only — swap for the project's framework primitives
import { OrderHttpRepository } from './structure/order-http-repository';
import { OrdersStore } from './store/orders.store';

const repository = new OrderHttpRepository(); // composition root for this feature
const store = new OrdersStore();

export function useOrdersFacade(customerId: string) {
  const [isLoading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);
  const [selectedId, setSelectedId] = useState<string | null>(null);

  async function load() {
    setLoading(true);
    try {
      store.setOrders(await repository.listByCustomer(customerId));
    } catch (e) {
      setError(e as Error);
    } finally {
      setLoading(false);
    }
  }

  return { orders: store.orders, isLoading, error, selectedId, selectOrder: setSelectedId, load };
}
```

### `pages/order-list.page.ts`

```ts
import { useOrdersFacade } from '../orders.facade';
import { OrderCardDumb } from '../components/order-card.dumb';

export function OrderListPage({ customerId }: { customerId: string }) {
  const { orders, isLoading, error, selectOrder } = useOrdersFacade(customerId);

  if (isLoading) return <Spinner />;
  if (error) return <ErrorBanner message={error.message} />;
  if (orders.length === 0) return <EmptyState />;

  return (
    <>
      {orders.map(order => (
        <OrderCardDumb key={order.id} order={order} onSelect={selectOrder} />
      ))}
    </>
  );
}
```

### `components/order-card.dumb.ts`

```ts
import type { Order } from '../domain/order.model';

interface OrderCardProps {
  order: Order;
  onSelect: (id: string) => void;
}

/** Props in, events out. No import from structure/, no fetch, no router. */
export function OrderCardDumb({ order, onSelect }: OrderCardProps) {
  return (
    <div onClick={() => onSelect(order.id)}>
      <span>{order.customerName}</span>
      <span>{order.total}</span>
    </div>
  );
}
```

### `orders.routes.ts`

```ts
// Framework-agnostic shape — adapt to the host router's API.
// The app shell imports this dynamically; it never defines orders' routes itself.
export const ordersRoutes = [
  { path: 'orders', component: () => import('./pages/order-list.page') },
];
```

### `index.ts`

```ts
export type { Order } from './domain/order.model';
export { OrderListPage } from './pages/order-list.page';
```

Only these two (plus `orders.routes.ts`, imported dynamically by the shell)
are ever imported from outside `features/orders/`. Everything else — the
repository, the mapper, the store, the facade — is internal.
