# Frontend Architecture — Antipatterns

Use this table in review/audit mode: walk the codebase, and for each match report `file:line → antipattern → replacement`.

| Antipattern | Why | Replacement |
| --- | --- | --- |
| HTTP call directly inside a component | couples the component to the wire format and to a specific fetch mechanism, untestable without a network | call a use case (`references/layers.md`) |
| DTO type used as a component prop or use-case return type | the wire shape leaks past the boundary that was supposed to contain it | map to a domain entity at the infrastructure mapper |
| `shared/utils.ts` (or similar) grab-bag file | nothing is provably dead, everything imports it "just in case" | split by actual consumer; promote to `shared/` only on a real third consumer |
| Deep import into another feature's internals (`features/orders/domain/order.entity`) from outside `orders/` | breaks "a feature is deletable by deleting its folder" | import through the feature's `index.ts` public entry |
| Domain entity decorated with a framework annotation (`@Injectable`, `@Component`, a framework base class) | domain becomes untestable without the framework's runtime | plain class, constructor-only dependencies |
| Use case returning the raw API response object | callers reimplement mapping ad hoc, inconsistently, per call site | map once, in infrastructure, return the entity |
| Barrel file importing from one of its own consumers | creates a circular import that surfaces unpredictably as the app grows | import the concrete module directly, not through the barrel |
| A "template" component that accepts a data-fetching prop or calls a hook with side effects | it's a page in disguise, defeating the reason templates exist | move the logic to the page, leave the template pure layout |
| Global `services/` folder holding every feature's API client | erases feature boundaries, everything can reach everything | one infrastructure folder per feature |
| Business rule expressed as a template/JSX conditional (`{order.status === 'confirmed' && order.paidAt && !order.refunded && ...}`) | untestable in isolation, silently duplicated at the next call site | a named method on the entity (`order.canBeCancelled()`) |
| Global mutable store holding both server data and UI toggles in the same slice | a UI toggle triggers unrelated re-fetch logic, or server data never gets invalidated | split server state and client state (`references/state.md`) |
| A dumb atom/molecule subscribing to a global store or calling a use case itself | can't be reused anywhere that store/use case doesn't exist | pass data and callbacks down from the container |
| Loading/error/empty branching duplicated inside every organism instead of once at the page | inconsistent states across the app, more code to keep in sync | branch once at the container, render fixed props below it |
| Entity's invariant re-implemented separately in a use case and in a component | the two copies drift the first time the rule changes | one method on the entity, called from both places |
| A `.repository.ts` file that both defines the interface and does the HTTP call | collapses domain's port and infrastructure's adapter into one file, so nothing enforces the inward dependency | split into `*.port.ts` (domain) and `*.adapter.ts` (infrastructure) |
| Feature folder with no `index.ts`, every file imported by its full internal path everywhere | no boundary between "this feature's public API" and its internals | add the barrel, migrate external imports through it |
| A store or hook named generically (`useData`, `store.ts`) with no suffix indicating its layer | can't tell from a search result whether it's server state, client state, or something else | name by role (`references/structure.md`'s suffix table) |
| Two features importing each other directly (mutual dependency) | neither can be deleted or extracted without the other, no clear ownership | extract the shared concern into `shared/` or `core/`, have both depend on that instead |
| Path aliases pointing at deep internal folders (`@features/orders/domain/*`) instead of the feature root | makes the illegal deep import as easy to write as the legal one | alias only the feature root; internals are reached through the barrel |
| An `atoms/` or `molecules/` folder inside a feature's `presentation/` | invites a near-duplicate `Button`/`Input` per feature instead of one shared primitive | move it to `shared/ui/atoms` or `shared/ui/molecules` — it isn't feature-bound |
| A `shared/ui` component typed against a domain entity or DTO (e.g. `{ order: Order }`) | makes the "shared" design system secretly depend on one feature's domain type | take a primitive/generic prop instead (`{ amount: number }`); if it truly needs the entity, it's an organism and belongs in the feature |
| A magic number/string compared directly (`order.statusCode === 'PEND'`) | forces every reader to already know what the literal means, and a typo compiles silently | name the constant or use an enum (`references/clean-code.md`) |
| A swallowed `catch {}`, or a `catch` that only logs and moves on | the failure disappears instead of being handled, and callers can't tell it happened | handle it or rethrow a named error |
| A function whose name doesn't mention a side effect it performs (`getUser()` that also clears a cache or fires analytics) | the name is a promise to the caller that the call is safe to make casually; breaking that promise causes surprise bugs | rename to say what it does, or split the read from the side effect |
| A single function mixing business rule, hashing/crypto, persistence, and I/O in one body | four different levels of abstraction in one place, hard to read and impossible to unit test in isolation | extract each concern behind its own function or port — one level of abstraction per function (`references/clean-code.md`) |
