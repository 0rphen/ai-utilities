# CSS Guidelines — Antipattern → Replacement Table

Use this table in review/audit mode: report each hit as `file:line → antipattern → replacement`.

| Antipattern | Why | Replacement |
| --- | --- | --- |
| `padding: 16px;` | not a relative unit | `padding-inline: var(--space-s);` (value confirmed with the requester) |
| `margin-top: 24px;` | physical property | `margin-block-start: var(--space-m);` |
| `color: #3366ff;` / `rgb(51,102,255)` / `oklch(0.6 0.2 260)` written in a block | color literal outside the tokens layer — not a token, can't be rethemed or shared | `color: var(--color-x);`, with the literal as that token's value in the tokens layer |
| `color: white;` / `background: black;` | named color, same problem — a literal outside the tokens layer | `color: var(--color-x);`, defined once in the tokens layer |
| `--brand-l: 0.6; --brand-c: 0.2; --brand-h: 260;` composed into `--color-brand` | stores channels that relative color syntax already reads off the token | one base token `--color-brand: oklch(0.6 0.2 260);`, channels via `oklch(from var(--color-brand) …)` (see `color.md`) |
| `@media (min-width: 900px)` sizing a single block/section that just happens to span the page | media query for something that's really a component concern, not a viewport concern | `container-type: inline-size;` + `@container` on that block |
| `.button:hover { background: oklch(0.72 0.2 250); }` | rewrites the whole triplet for a one-channel change | `.button:hover { background: oklch(from var(--color-x) calc(l + var(--l-step-hover)) c h); }` (see `color.md`) |
| `.button:hover { background: color-mix(in oklch, var(--color-x) 85%, white); }` | derives a state by mixing, shifts all channels unpredictably | derive with relative color syntax and a `-step-*` delta on the channel that actually changes |
| `oklch(var(--color-x) calc(l + 0.06) c h)` (missing `from`) | invalid syntax — `l`/`c`/`h` aren't in scope without `from`, the declaration is dropped silently | `oklch(from var(--color-x) calc(l + var(--l-step-hover)) c h)` |
| `oklch(from var(--color-x) calc(l + var(--l-step-hover)) c h)` with no clamp on an already-bright base | can push L out of `[0, 1]` or C negative, breaking the color | `clamp(0, calc(l + var(--l-step-hover)), 1)` for L, `max(0, calc(c + …))` for C |
| `:root { color-scheme: dark }` added preemptively before every surface has explicit color tokens | switches on the browser's UA dark defaults (canvas, form controls, unstyled text) anywhere `color`/`background` isn't explicit yet — breaks contrast silently | only set `color-scheme` once the real theme implementation covers every surface it affects |
| `light-dark(var(--color-x), var(--color-x-dark))` as the default dark-mode mechanism | needs a second token per color and only resolves once `color-scheme` is declared | redeclare `--color-x` under `prefers-color-scheme: dark` (or the project's theme selector) |
| `@media (max-width: 768px)` for an isolated component | media query where a container query belongs | `container-type: inline-size;` + `@container (width <= 48rem)` |
| Reordering elements in the DOM to change the mobile layout | couples structure to presentation | redefine `grid-template-areas` at the breakpoint |
| `!important` | breaks the cascade | move up a layer (`@layer`) or increase real specificity |
| A spacing value eyeballed from a mockup (`padding: 13px`) | assumes a measurement instead of confirming it | confirm the value with the requester, or use an existing token |
| A `-step-hover` delta invented on the spot ("looks about right") | still an unconfirmed value, same violation as any other assumed number | confirm it with the requester like any measurement |
| `.button {…}` followed by `.button:focus-visible {…}` as sibling rules | repeats the parent selector; the state no longer reads as part of the component | nest it: `.button { … &:focus-visible { … } }` |
| `&--primary` written bare (no leading `.`) inside a nested rule | not Sass — CSS nesting parses the identifier after `&` as a type selector, not a class suffix; compiles to a broken selector that matches nothing (esbuild: `Cannot use type selector "--primary" directly after nesting selector "&"`) | a CUBE exception on `[data-variant='primary']` (nests cleanly, no suffix to break) — or, if the project already uses BEM/SMACSS, `&.button--primary`/`&.is-active` (full class, verified same DOM node) |
| Nesting more than 2 levels with `&` | excessive nesting, hard to read | concatenate an exception's state at level 2 (`&[data-variant='primary']:hover`); if it still needs more, flatten into its own block at the corresponding CUBE layer |
| `flex` with percentage widths + a media query for a uniform gallery | reimplements what grid does natively | `repeat(auto-fit, minmax(min(100%, <min>), 1fr))` |
| `margin-inline-start: auto` / a spacer element to push an item | margin hack instead of real alignment | `justify-content` / `align-items` on the container |
| `margin` between siblings to space them out | couples spacing to sibling order | `gap` on the flex/grid container |
| `padding-block-start: 56.25%` (padding-top proportion hack) | fragile, unitless-looking magic number | `aspect-ratio: 16 / 9` |
| fixed `height`/`block-size` to hold a media element's proportion | breaks at other sizes | `aspect-ratio` + `object-fit` |
| `grid-template-areas` forced onto a collection of N equal items | areas are for named/heterogeneous zones, not a repeated set | `repeat(auto-fit or auto-fill, minmax(...))` |
| `--_radius: var(--radius-m); border-radius: var(--radius-m);` | internal declared but never consumed — dead indirection, the extension point doesn't really exist | `border-radius: var(--_radius);` |
| `.button:hover { background-color: oklch(from …) }` repeating the property in every state of a block that already has internals | duplicates the final declaration; the configurable axis stops being singular | reassign `--_bg` in the state, with `background-color: var(--_bg)` declared once (see `color.md`) |
| `.sidebar .card { --_radius: var(--radius-s) }` | reassigns an internal from outside its block and crosses two classes in one selector | nested exception `&[data-variant='…']` inside `.card` |
| `@container style(--density: compact) { .card { padding: 0.75rem; background: #fff; } }` | literals inside the query, and final properties redeclared per context instead of internals reassigned | nest the query inside `.card` and reassign `--_pad-block`/`--_bg` from tokens; declare `padding-block: var(--_pad-block)` once (see `layout.md`) |
| `.l-grid-compact .card { … }` to adapt a block to its container's mode | descendant selector reaching into the block from outside, crossing two classes | the container sets a context property; the block reads it with a nested `@container style(--x: value)` |
| `style="--density: compact"` in markup, or a context property set from the block layer | the context escapes the composition layer — untraceable, not themeable | set it on the `.l-*` container in the composition layer (e.g. under `&[data-density='compact']`) |
| `.card { --density: compact; @container style(--density: compact) { … } }` | a style query evaluates an ancestor, never the element itself — it never matches | set the context property on an ancestor container |
| `--_bg: var(--color-brand);` on a block where no state, variant, or theme moves that axis | internal with nothing to vary — reuse alone doesn't justify the indirection | consume `var(--color-brand)` directly (see `tokens.md`) |
