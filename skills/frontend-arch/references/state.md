# Frontend Architecture — State

Read this before adding a store, a cache, or deciding which layer owns a new piece of state.

## Four Kinds of State, Four Different Lifecycles

| Kind | Example | Owned by | Lifecycle |
| --- | --- | --- | --- |
| Server state | The order list, a user profile | Application, behind a use case | Fetched, can go stale, can be shared across screens, invalidated on writes. |
| URL state | Current page number, active filter, selected tab-as-route | The router | Shareable via link, back/forward-navigable, source of truth for anything that should survive a refresh. |
| Client/UI state | A dropdown's open/closed, a form draft, a wizard step | Presentation, local to the component/page that needs it | Never persisted, never fetched, dies with the component unless deliberately lifted. |
| Ephemeral derived state | "Is this order cancellable right now" | Computed from server state, not stored separately | Recomputed on every render from the entity, never cached as its own field. |

Treating all four as one undifferentiated "app state" blob in a single global store is the most common source of state bugs: a UI toggle triggers a refetch because it lives next to server data in the same slice, or server data goes stale because nothing knows it needs revalidating once it's sitting next to state that never expires.

## Server State Is an Application-Layer Concern

A server-state store (whatever the mechanism — a query-cache library, a plain store with a use case behind it) is populated by calling a use case, not by a component reaching into infrastructure directly:

```typescript
// order-list.store.ts — application layer, framework-agnostic shape
export class OrderListStore {
  private orders: Order[] = [];
  private loading = false;

  constructor(private readonly listOrders: ListOrdersUseCase) {}

  async load(): Promise<void> {
    this.loading = true;
    this.orders = await this.listOrders.execute();
    this.loading = false;
  }
}
```

The store's job is caching, invalidation, and loading/error bookkeeping around a use case call — it is not a place to reimplement fetching logic that duplicates what the use case already does.

## Client State Stays in Presentation

A form draft, a collapsed/expanded flag, a hover state — these have no reason to exist once the component unmounts and no reason to be reachable from another feature. They live as local component state (whatever primitive the framework offers), not in a global store, unless a concrete second consumer needs to read them.

## Loading / Error / Empty Boundaries Belong at the Container

The page (or a deliberately-smart organism) is what checks "is this still loading / did this error / is this empty" and picks which child to render — a dumb atom or molecule never branches on those states itself, it just renders what it's handed:

```typescript
// order-detail.page.tsx
function OrderDetailPage({ orderId }: { orderId: string }) {
  const { order, loading, error } = useOrderStore(orderId);

  if (loading) return <Spinner />;
  if (error) return <ErrorState message={error.message} />;
  if (!order) return <EmptyState />;

  return <OrderSummaryCard order={order} onCancel={cancelOrder} />;
}
```

This keeps every organism/molecule/atom below the page trivially testable with a fixed prop, with no need to simulate a loading or error state through a store.

## URL State Is the Router's, Not a Duplicate Store Field

If a value should survive a page refresh or be shareable via a link (a filter, a sort order, a selected ID), it belongs in the URL, read through the router — not mirrored into a store field that can drift out of sync with the address bar.
