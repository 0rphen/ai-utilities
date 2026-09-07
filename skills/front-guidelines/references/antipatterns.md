# Anti-patterns

| Anti-pattern | Fix |
|---|---|
| Global `components/`/`services/`/`models/` at app root | Move under owning feature, or `core`/`shared`/`utils` |
| An implementation (HTTP call, mapper logic) placed inside `domain/` | Move it to `structure/`; `domain/` stays declarations only |
| Repository port only implicit (inferred from `structure/`), no interface/abstract class in `domain/` | Add the explicit port to `domain/` first |
| DTO typed into `pages/`/`components/`/`store/`/`domain/` | Add the missing mapper in `structure/` |
| "Dumb" component importing `structure/` or fetching | Lift logic to the facade, consumed by `pages/` |
| `pages/` importing `structure/` or `store/` directly, bypassing the facade | Route everything through the facade |
| Skipping the facade because a feature has only one page | Keep it — the facade is mandatory regardless of consumer count |
| Cross-feature deep import | Export via `index.ts`, or move to `shared/`/`utils/` |
| Cache library (React Query, SWR, `resource()`) inside `structure/` | Move to the facade; keep `structure/` framework-free |
| Hardcoded style value where a token exists | Use (or add) the token |
| Domain-aware component placed in `shared/` | Move it into the owning feature's `components/`; `shared/` holds no domain knowledge |
| Stateful or UI-aware helper placed in `utils/` | Move it to `core/` (if singleton) or the feature; `utils/` is pure functions only |
| Store passed down as a prop/input | Obtain the store via the facade instead |
| External code mutating store state directly (`store.orders.push(...)`) | Expose only readonly state + named methods (`store.addOrder(...)`) |
| Derived value recomputed in the render/template on every read | Compute it once inside the store as a derived/computed value |
| Effect that writes back to the state it depends on | Use an explicit method call instead; keep effects one-directional (outgoing side-effects only) |
| Store calling the repository directly, or holding a DTO | Route through the facade; store holds models/UI-shaped data only |
| A repository/facade/component doing more than its one job (e.g. a repository also formatting UI strings) | Split by responsibility (SRP) — one file, one job |
| Code depending on the concrete `structure/` implementation instead of the `domain/` port | Depend on the abstract type; inject/import the port, not the implementation (DIP) |
| New behavior added by branching inside an existing repository/mapper instead of a new implementation | Add a new implementation of the port and swap it in (OCP) |
| The same mapping/validation/formatting rule copy-pasted in two or more features | Lift it into `domain/` (business rule) or `shared/`/`utils/` (generic) and reuse it (DRY) |
