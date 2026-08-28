# Frontend Architecture — Presentation

Read this before decomposing a UI, splitting an oversized component, or deciding whether a piece of markup should hold logic.

## The Problem This Solves

A "smart component" can grow to hundreds of lines of logic and a comparable amount of markup, becoming harder to touch with every addition even when no single piece of logic is complex — the length alone is the problem. The smart/dumb (container/presenter) split addresses *who owns logic*, but says nothing about *how the markup itself* should be structured. Atomic design fills that second axis.

## Two Orthogonal Axes

**Atomic level** — how composed a piece of UI is:

| Level | Definition | Lives in |
| --- | --- | --- |
| Atom | Smallest reusable unit — one element with baseline styling (`Button`, `Input`, `Badge`). | `shared/ui/atoms` |
| Molecule | A small group of atoms with a single purpose (`SearchField` = `Input` + `Button`). | `shared/ui/molecules` |
| Organism | A self-contained, non-trivial section — a "widget" that could be dropped into more than one page and still make sense on its own (`OrderSummaryCard`, `NavigationBar`). | `features/*/presentation/organisms` |
| Template | Pure layout scaffold: regions, slots, responsive structure, and styling — nothing else. | `features/*/presentation/templates` |
| Page | The route-level component that owns data fetching (via use cases) and composes everything below it. | `features/*/presentation/pages` |

**Smartness** — who owns logic and calls out to the application layer: **smart** components call use cases, hold state derived from them, and pass data down; **dumb** components receive props/inputs and emit events/callbacks, nothing more.

These axes are independent. An atom is always dumb. A page is always smart. But an organism can be either — a self-contained `LiveOrderStatusWidget` that subscribes to its own use case is a smart organism; an `OrderSummaryCard` that just renders the order it's given is a dumb one. There's no rule of thumb for which — keep logic in the page whenever that's simple, and only push a use-case call down into an organism when the page has genuinely grown too large to stay flat.

## Where Each Level Lives

Atoms and molecules are the reusable end of the ladder — that's the whole reason to name them separately from an organism. Giving each feature its own `atoms/` folder undoes that: a `Button` copied into `features/orders` and again into `features/suppliers` isn't an atom anymore, it's two drifting near-duplicates. So the rule is unconditional, not a judgment call: **every atom and every molecule lives in the shared design system (`shared/ui`); a feature's `presentation/` folder holds only organisms, templates, and pages.**

That rule needs a test that doesn't rely on taste, because "is this reusable enough to be shared" invites debate. The test is the prop, not a feeling about the component: **a shared design-system component takes primitives and generic shapes only — `{ amount: number }`, `{ label: string, disabled: boolean }` — never a domain entity or DTO.** The moment a component needs an `Order` or a `Producto` as a prop to make sense, it has domain vocabulary baked into it, which means it isn't a shared primitive anymore — it's an organism, and it belongs in the feature that owns that entity, no matter how small it is.

## The Templates Rule

Templates are the one atomic level with a hard constraint: **no props beyond content/children, no events, no logic, no injected dependencies — scaffold and skin only.**

<!-- ✅ -->
```typescript
// order-page.template.tsx — layout and styling, nothing else
export function OrderPageTemplate({ header, summary, actions }: {
  header: ReactNode; summary: ReactNode; actions: ReactNode;
}) {
  return (
    <div class="l-order-page">
      <header class="l-order-page__header">{header}</header>
      <main class="l-order-page__summary">{summary}</main>
      <footer class="l-order-page__actions">{actions}</footer>
    </div>
  );
}
```

<!-- ❌ never — a template calling a use case or branching on business state -->
```typescript
// order-page-bad.template.tsx
export function OrderPageTemplateBad({ orderId }: { orderId: string }) {
  const order = useOrder(orderId); // logic — this makes it a page, not a template
  if (!order) return <Spinner />;
  return <div class="l-order-page">{/* ... */}</div>;
}
```

A component satisfying the real templates rule can be refactored out of a smart page in minutes — it's a cut-and-paste of markup and styling with the dynamic pieces replaced by children/slots. If that refactor turns out to require moving logic too, the thing being extracted was a page fragment, not a template.

## Why Organisms Are Worth Naming Separately

Without an organism level, a page either stays one flat file or gets arbitrarily split into molecules that are really "half a feature" in disguise. Naming the organism level explicitly — self-contained widget, reusable across more than one page, non-trivial but not the whole screen — gives a natural stopping point for decomposition instead of splitting forever or not at all.

## Do / Don't

<!-- ✅ -->
```typescript
// order-summary-card.organism.tsx — dumb: renders what it's given
export function OrderSummaryCard({ order, onCancel }: { order: Order; onCancel: (id: string) => void }) {
  return (
    <article class="c-order-card">
      <PriceTag amount={order.total} />
      {order.canBeCancelled() && <Button onClick={() => onCancel(order.id)}>Cancel</Button>}
    </article>
  );
}
```

<!-- ❌ never — two ways an atom stops being reusable -->
```typescript
// price-tag-bad.atom.tsx
export function PriceTagBadA({ orderId }: { orderId: string }) {
  const order = useOrderStore((s) => s.orders[orderId]); // reaches into a store — can't reuse where the store doesn't exist
  return <span class="c-price">{order.total}</span>;
}

export function PriceTagBadB({ order }: { order: Order }) { // Order is the orders feature's entity
  return <span class="c-price">{formatMoney(order.total)}</span>; // typed against a domain entity — drags the feature with it
}
```

Keep the async/store boundary at the container (page, or a deliberately-smart organism), and keep the prop primitive:

<!-- ✅ -->
```typescript
// shared/ui/atoms/price-tag.atom.tsx — primitive prop, works for any feature
export function PriceTag({ amount }: { amount: number }) {
  return <span class="c-price">{formatMoney(amount)}</span>;
}
```

Pass `amount={order.total}` from the organism that has the entity instead of importing the entity type into the primitive.
