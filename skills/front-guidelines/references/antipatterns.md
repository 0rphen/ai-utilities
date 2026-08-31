# Anti-patterns

| Anti-pattern | Fix |
|---|---|
| Global `components/`/`services/`/`models/` at app root | Move under owning feature, or `core`/`shared` |
| Fat repository (interface + HTTP call in one file) | Split into `*.repository.port.ts` (domain) + implementation (data) |
| DTO typed into `ui/`/`domain/` | Add the missing mapper |
| "Dumb" component importing `data/` or fetching | Lift logic to smart/facade |
| Cross-feature deep import | Export via `index.ts`, or move to `shared/` |
| Cache library (React Query, SWR, `resource()`) inside `data/` | Move to `ui/`; keep `data/` framework-free |
| Hardcoded style value where a token exists | Use (or add) the token |
| `ui/` calling a datasource directly, skipping the repository (even in `small` tier) | Always go through the repository — the tier only decides whether the repository has a separate datasource file, never whether `ui/` can bypass it |
| Store passed down as a prop/input | Obtain the store via DI/import/context instead |
| External code mutating store state directly (`store.orders.push(...)`) | Expose only readonly state + named methods (`store.addOrder(...)`) |
| Derived value recomputed in the render/template on every read | Compute it once inside the store as a derived/computed value |
| Effect that writes back to the state it depends on | Use an explicit method call instead; keep effects one-directional (outgoing side-effects only) |
| Store calling a datasource directly, or holding a DTO | Route through the repository; store holds entities/UI-shaped data only |
| TTL/offline-fallback policy implemented in `ui/` | Move the policy into the repository, orchestrating `remote` + `local` datasources |
