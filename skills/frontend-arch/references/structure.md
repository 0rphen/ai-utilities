# Frontend Architecture — Structure

Read this before creating folders for a new feature, naming a new file, or wiring lint-enforced import boundaries.

## Feature-First Folder Tree

```
src/
  app/                    → composition root: routing, DI wiring, global providers
  core/                   → cross-feature infrastructure with no UI: http client setup, auth session, logging
  shared/
    ui/                   → the design system: every atom and molecule in the app lives here, never inside a feature
      atoms/              → Button, Input, Badge, Spinner, PriceTag
      molecules/          → SearchField, EmptyState
  features/
    orders/
      domain/             → entities, value objects, domain errors, repository/gateway ports
      application/        → use cases, one file per operation
      infrastructure/     → DTOs, mappers, adapters implementing the domain ports
      presentation/
        organisms/        → OrderSummaryCard (may take a domain entity — that's what makes it an organism, not an atom)
        templates/        → OrderPageLayout (scaffold + styling only, see references/presentation.md)
        pages/            → OrderDetailPage (smart — calls the use cases)
      index.ts            → the feature's public entry: re-exports what other features may use
    suppliers/
      domain/
      application/
      infrastructure/
      presentation/
      index.ts
```

Atoms and molecules are the *reusable* end of the atomic ladder (see `references/presentation.md`) — giving each feature its own `atoms/` folder just grows a near-duplicate `Button` per feature, which is the exact outcome atomic design exists to prevent. They live in `shared/ui` unconditionally; a feature's `presentation/` only ever holds organisms, templates, and pages.

A feature should be deletable by deleting its folder. If deleting `features/orders/` would break `features/suppliers/`, something imported past `orders/index.ts` into `orders/`'s internals — that's the violation Instruction 12's lint rule exists to catch.

## File-Suffix Naming Scheme

A suffix should make a file's layer identifiable from a flat search result (`**/*.usecase.ts`) without opening it or knowing which folder it's in:

| Suffix | Layer | Example |
| --- | --- | --- |
| `.entity.ts` | domain | `order.entity.ts` |
| `.value-object.ts` | domain | `money.value-object.ts` |
| `.port.ts` | domain (interface, implemented outward) | `order-repository.port.ts` |
| `.usecase.ts` | application | `cancel-order.usecase.ts` |
| `.dto.ts` | infrastructure | `order.dto.ts` |
| `.mapper.ts` | infrastructure | `order.mapper.ts` |
| `.adapter.ts` / `.repository.ts` (impl) | infrastructure | `http-order-repository.adapter.ts` |
| `.store.ts` | application or presentation, see `references/state.md` | `order-list.store.ts` |

Don't invent a suffix scheme per project unless there's already a strong house convention — adopt whatever the codebase already uses over this table (Instruction 1).

## Barrel-File Policy

- One `index.ts` per feature, at the feature root, re-exporting only what's meant to be public (use cases another feature legitimately calls, types another feature legitimately needs, top-level page components for routing).
- No barrel file inside a layer folder (`domain/index.ts`, `presentation/organisms/index.ts`) unless the project's bundler needs one for tree-shaking reasons — an extra barrel is another place a circular import can hide.
- Nothing inside a feature imports its own `index.ts` — that's the exact shape that creates a cycle once anything in the barrel re-exports something that (transitively) imports the barrel back.

## Path Aliases

Alias each top-level bucket so an import states its layer instead of a relative-path crawl:

```
@app/*       → src/app/*
@core/*      → src/core/*
@shared/*    → src/shared/*
@features/*  → src/features/*
```

`import { CancelOrderUseCase } from '@features/orders'` (through the barrel) is legitimate; `import { Order } from '@features/orders/domain/order.entity'` from outside the feature is not — the alias makes both the intended and the violating import equally easy to `grep` for.

## Test Colocation

`cancel-order.usecase.ts` and `cancel-order.usecase.spec.ts` sit in the same folder. A parallel `__tests__/` or `test/` tree mirroring `src/` drifts: files get renamed or deleted on one side and not the other, and nothing catches it until someone notices coverage silently dropped.

## Enforcing Boundaries With a Lint Rule

A dependency-direction rule nobody enforces mechanically gets violated within a sprint under deadline pressure. Sketch, using `eslint-plugin-boundaries` (any equivalent — `import/no-restricted-paths`, a custom ESLint rule, a dependency-cruiser config — works the same way):

```javascript
// .eslintrc — sketch, adapt element types/paths to the project
module.exports = {
  plugins: ['boundaries'],
  settings: {
    'boundaries/elements': [
      { type: 'domain', pattern: 'src/features/*/domain/*' },
      { type: 'application', pattern: 'src/features/*/application/*' },
      { type: 'infrastructure', pattern: 'src/features/*/infrastructure/*' },
      { type: 'presentation', pattern: 'src/features/*/presentation/*' },
      { type: 'design-system', pattern: 'src/shared/ui/*' },
    ],
  },
  rules: {
    'boundaries/element-types': ['error', {
      default: 'disallow',
      rules: [
        { from: 'domain', allow: [] },
        { from: 'application', allow: ['domain'] },
        { from: 'infrastructure', allow: ['domain', 'application'] },
        { from: 'presentation', allow: ['domain', 'application', 'design-system'] },
        { from: 'design-system', allow: [] },
      ],
    }],
  },
};
```

`from: 'domain', allow: []` is the mechanical version of Instruction 3 — a domain file that imports anything outside domain fails CI, not just review. `from: 'design-system', allow: []` is the same idea applied to the shared UI layer: a component in `shared/ui` that imports a domain entity has quietly made the design system depend on a feature, which defeats the point of it being shared.
