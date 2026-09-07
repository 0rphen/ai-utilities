# Anti-patterns

| Anti-pattern | Fix |
|---|---|
| Observable used merely to represent ordinary UI/application state | Use a signal; reserve RxJS for stream-shaped flows |
| Effect writing back to the state it depends on | Use an explicit method call; keep effects one-directional (outgoing side effects only) |
| Effect used as a general state-sync mechanism | Trigger sync from the state change site, not from an effect reacting to it |
| HTTP client, URL, or transport error reaching a component | Route through the repository; map errors at the data-access boundary |
| DTO typed into `domain/`, `store/`, `components/`, or `pages/`, or reaching a template | Add the missing mapper at the `structure/`↔`domain` boundary |
| `resource()`/`httpResource()` inside `structure/` | Move to the facade; keep `structure/` free of presentation/runtime primitives |
| `pages/` calling `structure/` or `store/` directly, bypassing the facade | Route everything through the facade |
| Store promoted to app-wide scope for convenience, without 2+ consumers or cross-navigation need | Keep it feature-scoped until the trigger actually appears |
| Derived value duplicated in a writable signal instead of `computed()` | Compute it once inside the store/component as a derived value |
| Injectable accumulating unrelated responsibilities (a "god service") | Split by responsibility; keep each service coherent |
| Interface or injection token introduced without a substitution/testability reason | Depend on the concrete class directly until one appears |
| Business rule embedded in a template | Move it to a pure `domain/` service |
| Component extracted with no reuse or clarity gain | Inline it back; extract only on a concrete need |
| Architecture driven by folder/feature naming rather than responsibility | Classify each piece by what it does, not what it's called |
| A version-specific API assumption stated as an architectural rule | State the principle version-independently; verify the primitive against the installed version separately |
