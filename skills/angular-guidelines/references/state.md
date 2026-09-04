# State

Where state lives and how it's shaped in Angular terms. See `SKILL.md` rules
5–8 and `front-guidelines`' `references/state.md` for the framework-agnostic
ladder this mirrors.

## The ladder

1. **Component-local** — `signal()` directly in the component. Starting point,
   always.
2. **Feature store** — `@Injectable({ providedIn: 'root' })` or route-scoped,
   with a private writable signal. Promote on a concrete trigger: a second
   consumer needs the same state, or a derived value would otherwise be
   recomputed and drift.
3. **App-wide store** — same shape, placed where the project's runtime
   singletons live (session, theme, feature flags).
4. **Store library** — bring in a dedicated signal-based store solution only
   once app-wide state has real cross-feature interdependencies and the team
   needs devtools/time-travel. Don't reach for this to manage one feature's
   state — level 2 already covers that.

## Store contract

```ts
@Injectable({ providedIn: 'root' })
export class CartStore {
  #items = signal<CartItem[]>([]);
  items = this.#items.asReadonly();

  totalItems = computed(() => this.#items().reduce((n, i) => n + i.quantity, 0));

  addItem(item: CartItem) {
    this.#items.update(current => [...current, item]);
  }
}
```

Verify the exact primitive names (`signal`, `computed`, `asReadonly`, the
injection form) against the project's installed Angular version before
generating this — the shape is stable, the API surface isn't guaranteed to be.

- Private mutable signal, public readonly accessor.
- Derived values are `computed()`, never recomputed in a template.
- Mutation only through named methods — never a raw setter.
- The store is obtained via DI, never passed as a prop/input to a dumb
  component.

## Signals vs RxJS

Use signals for application state. Reach for RxJS when the flow needs
stream-oriented semantics: composition, cancellation, concurrency, buffering,
polling, continuous event streams. Convert between them at an explicit
boundary. Never wrap ordinary state in an observable just to have one; never
replace a flow that genuinely needs RxJS with signal orchestration.

## Effects

`effect()` is for outgoing side effects only — persistence, logging, syncing
to another system. Never let an effect write back to the same state it reads:
that's a feedback loop, not a side effect. This is easy to introduce by
accident when an effect triggers an async operation (e.g. a sync retry) whose
success handler then writes to the state the effect depends on — audit that
path explicitly whenever an effect drives a network call.

## A store is not a repository

A store calls the repository for anything leaving the process — it never
calls a datasource or HTTP client directly, and it never holds a DTO. It holds
entities or UI-shaped projections of them, same mapper boundary as
`references/boundaries.md`.
