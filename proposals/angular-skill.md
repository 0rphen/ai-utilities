# Proposal: `angular-developer-guidelines` skill

Not a skill — a draft. This directory holds proposals, not installable
content; nothing here is loaded by Claude Code. Extracted from a user-
supplied doc (`arquitectura-angular-22-screaming-4.md`, Angular 22, June
2026) while building tiered scaling into `skills/front-guidelines`. Kept
because it's genuinely useful for a *future*, Angular-specific skill layered
on top of `front-guidelines` — but it doesn't belong in a framework-agnostic
skill, and the source doc has real bugs that a future author must not carry
forward as-is (flagged below).

## Relationship to `front-guidelines`

This would be a companion skill, not a replacement: `front-guidelines`
supplies the tier ladder, the `domain/data/ui` layering, the port/mapper
contract, and the smart/dumb split; this proposal supplies how those map to
Angular 22 primitives specifically. Any future skill built from this must
follow `front-guidelines`' folder shape (`domain/data/ui`, not the source
doc's `data-access/feature/ui`) and its English-only code rule — the source
doc is in Spanish and uses its own tree.

## Platform baseline (Angular 22)

- Standalone-only — no NgModules in new code.
- `provideZonelessChangeDetection()` + `OnPush` by default.
- Vite/esbuild; Webpack path deprecated.
- HTTP client on Fetch API by default.
- TypeScript v6+, Node v26+ required.

## Tier → Angular primitive mapping

Reuses `front-guidelines`' `small`/`medium`/`large` tiers (see
`skills/front-guidelines/references/tiers.md`) rather than inventing a
separate scale:

- **Data fetching**: `httpResource()`/`resource()` lives in `ui/`, wrapping a
  call to the repository — never replacing it, never appearing in `data/`
  (same rule as `front-guidelines`' cache-library guidance).
- **State ladder** (mirrors `skills/front-guidelines/references/state.md`):
  - L1 (component-local): `signal()` directly in the component.
  - L2 (feature store): `@Injectable({ providedIn: 'root' })` class with a
    private `#state = signal(...)` and `state = this.#state.asReadonly()`.
  - L3 (app-wide store): same shape, placed in `core/` (session, theme).
  - L4 (`large` only): NgRx Signal Store (`signalStore`, `withState`,
    `withComputed`, `withMethods`, `patchState`) for interdependent app-wide
    state needing devtools/time-travel.
- **Routing**: `loadChildren` per feature, composed in `app.routes.ts`;
  matches `front-guidelines` rule 11 (feature declares its own routes, shell
  only composes/lazy-loads).
- **Bootstrap**: `bootstrapApplication()` with
  `provideZonelessChangeDetection()`, `provideRouter(routes)`,
  `provideHttpClient()`.
- **Interceptors/guards**: `core/interceptors/`, `core/guards/` — matches the
  existing `core/` criterion (single runtime instance, no domain logic).

## Store contract in signals (Angular-specific version of `state.md`)

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

Same rules as `state.md`: private mutable signal, public `asReadonly()`,
`computed()` for derived values (never recomputed in the template),
mutation only through named methods, `effect()` reserved for outgoing
side-effects (e.g. persistence) and never writing back to its own
dependency.

## Bugs in the source doc — do not carry forward

The source doc's code samples have real defects; a future skill author
should fix these, not copy them:

1. **Invalid `withMethods` signature.** The doc writes
   `withMethods((store, private http = inject(HttpClient)) => ({ ... }))` —
   `private` is not valid in a plain arrow-function parameter list (that
   syntax only exists in a constructor parameter). The correct form is
   `withMethods((store) => { const http = inject(HttpClient); return { ... }; })`.
2. **`this.http` used inside those closures.** Since the parameter above
   isn't a real class member, `this.http`/`this.messageService` inside the
   returned methods don't resolve to anything — same fix as above (capture
   via a local `const` in the factory's closure, not `this`).
3. **`CartSyncService`'s `effect()` risks a feedback loop.** It reads
   `this.cart.items()` and calls `attemptSync()`, which itself doesn't write
   `items` — that part is fine — but the pattern as written conflates
   "effect reacts to state" with "effect drives a network call with retry
   logic inline," which is easy to accidentally turn into a cycle if a
   future edit has the sync success handler write back to the same signal
   the effect reads. Flag this explicitly in the future skill's own
   anti-patterns table (mirroring `front-guidelines`' "effect that writes
   back to the state it depends on" row).

## Explicitly out of scope for this proposal

Everything `front-guidelines` already owns and is framework-agnostic:
screaming/feature-first structure, `domain/data/ui` layering, port + mapper
contract, hard smart/dumb split, `core`/`shared` criterion, tier ladder
itself. This proposal only adds the Angular-specific "how," never a
second, competing "what."
