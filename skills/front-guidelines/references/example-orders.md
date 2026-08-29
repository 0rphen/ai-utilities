# Canonical example: `orders`

Framework-agnostic TypeScript. This is the template to adapt — file names,
folder shape, and layer boundaries transfer as-is; syntax adapts to whatever
framework the project uses.

```
features/orders/
  domain/
    order.entity.ts
    order.repository.port.ts
    order-pricing.service.ts
  data/
    order.dto.ts
    order.mapper.ts
    order.datasource.port.ts
    order.remote.datasource.ts
    order.repository.ts
  ui/
    orders.facade.ts
    order-list.smart.ts
    order-card.dumb.ts
  index.ts
```

### `domain/order.entity.ts`

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

### `domain/order.repository.port.ts`

```ts
import type { Order } from './order.entity';

export interface OrderRepository {
  findById(id: string): Promise<Order>;
  listByCustomer(customerId: string): Promise<Order[]>;
}
```

### `domain/order-pricing.service.ts`

```ts
import type { Order } from './order.entity';

/** Pure business rule: no I/O, no framework import. */
export function applyBulkDiscount(order: Order): number {
  const itemCount = order.items.reduce((sum, i) => sum + i.quantity, 0);
  const discount = itemCount >= 10 ? 0.1 : 0;
  return order.total * (1 - discount);
}
```

### `data/order.dto.ts`

```ts
export interface OrderDto {
  id: string;
  customer_name: string;
  items: { sku: string; qty: number; unit_price: number }[];
  total_amount: string; // API returns a string
  placed_at: string;    // ISO date string
}
```

### `data/order.mapper.ts`

```ts
import type { Order } from '../domain/order.entity';
import type { OrderDto } from './order.dto';

export const OrderMapper = {
  toEntity(dto: OrderDto): Order {
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

### `data/order.datasource.port.ts`

```ts
import type { OrderDto } from './order.dto';

export interface OrderRemoteDatasource {
  fetchById(id: string): Promise<OrderDto>;
  fetchByCustomer(customerId: string): Promise<OrderDto[]>;
}
```

### `data/order.remote.datasource.ts`

```ts
import type { OrderRemoteDatasource } from './order.datasource.port';
import type { OrderDto } from './order.dto';
import { httpClient } from '../../../core/http/http-client';
import { OrderNotFoundError, OrderFetchError } from '../domain/order.errors';

export class OrderRemoteDatasourceImpl implements OrderRemoteDatasource {
  async fetchById(id: string): Promise<OrderDto> {
    const res = await httpClient.get(`/orders/${id}`);
    if (res.status === 404) throw new OrderNotFoundError(id);
    if (!res.ok) throw new OrderFetchError(res.status);
    return res.json();
  }

  async fetchByCustomer(customerId: string): Promise<OrderDto[]> {
    const res = await httpClient.get(`/customers/${customerId}/orders`);
    if (!res.ok) throw new OrderFetchError(res.status);
    return res.json();
  }
}
```

### `data/order.repository.ts`

```ts
import type { OrderRepository } from '../domain/order.repository.port';
import type { Order } from '../domain/order.entity';
import type { OrderRemoteDatasource } from './order.datasource.port';
import { OrderMapper } from './order.mapper';

export class OrderRepositoryImpl implements OrderRepository {
  constructor(private readonly remote: OrderRemoteDatasource) {}

  async findById(id: string): Promise<Order> {
    const dto = await this.remote.fetchById(id);
    return OrderMapper.toEntity(dto);
  }

  async listByCustomer(customerId: string): Promise<Order[]> {
    const dtos = await this.remote.fetchByCustomer(customerId);
    return dtos.map(OrderMapper.toEntity);
  }
}
```

### `ui/orders.facade.ts`

```ts
import { useState } from 'react'; // example only — swap for the project's framework primitives
import { useQuery } from '@tanstack/react-query';
import { orderRepository } from './orders.wiring'; // composition root for this feature

export function useOrdersFacade(customerId: string) {
  const { data, isLoading, error } = useQuery(
    ['orders', customerId],
    () => orderRepository.listByCustomer(customerId),
  );
  const [selectedId, setSelectedId] = useState<string | null>(null);

  return { orders: data ?? [], isLoading, error, selectedId, selectOrder: setSelectedId };
}
```

### `ui/order-list.smart.ts`

```ts
import { useOrdersFacade } from './orders.facade';
import { OrderCardDumb } from './order-card.dumb';

export function OrderListSmart({ customerId }: { customerId: string }) {
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

### `ui/order-card.dumb.ts`

```ts
import type { Order } from '../domain/order.entity';

interface OrderCardProps {
  order: Order;
  onSelect: (id: string) => void;
}

/** Props in, events out. No import from data/, no fetch, no router. */
export function OrderCardDumb({ order, onSelect }: OrderCardProps) {
  return (
    <div onClick={() => onSelect(order.id)}>
      <span>{order.customerName}</span>
      <span>{order.total}</span>
    </div>
  );
}
```

### `index.ts`

```ts
export type { Order } from './domain/order.entity';
export { OrderListSmart } from './ui/order-list.smart';
```

Only these two are ever imported from outside `features/orders/`. Everything
else — the repository, the datasource, the mapper, the facade — is internal.
