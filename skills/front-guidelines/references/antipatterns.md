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
