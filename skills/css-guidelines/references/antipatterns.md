# CSS Guidelines — Antipattern → Replacement Table

Use this table in review/audit mode: report each hit as `file:line → antipattern → replacement`.

| Antipattern | Why | Replacement |
| --- | --- | --- |
| `padding: 16px;` | not a relative unit | `padding-inline: var(--space-s);` (value confirmed with the requester) |
| `margin-top: 24px;` | physical property | `margin-block-start: var(--space-m);` |
| `color: #3366ff;` / `rgb(51,102,255)` | not oklch | `color: oklch(var(--x-l) var(--x-c) var(--x-h));` with channel tokens |
| `color: white;` / `background: black;` | named color, same problem as hex — not a token, not composable into a state | `color: oklch(var(--x-l) var(--x-c) var(--x-h));` with channel tokens |
| `@media (min-width: 900px)` sizing a single block/section that just happens to span the page | media query for something that's really a component concern, not a viewport concern | `container-type: inline-size;` + `@container` on that block |
| `.button:hover { background: oklch(0.72 0.2 250); }` | rewrites the whole triplet for a one-channel change | `.button:hover { --x-l: calc(var(--x-l-base) + var(--l-step-hover)); }` (see `color.md`) |
| `.button:hover { background: color-mix(in oklch, var(--color-x) 85%, white); }` | derives a state by mixing, shifts all channels unpredictably | override the live channel(s) directly with a `-step-*` delta |
| `--x-l: calc(var(--x-l) + 0.06);` | self-referential — cycles, property goes guaranteed-invalid | `--x-l: calc(var(--x-l-base) + var(--l-step-hover));` |
| `--color-x` composed once at `:root`, then `.button:hover { --x-l: ... }` expecting `background: var(--color-x)` to recompute | `--color-x` is only *specified* at `:root` — descendants inherit the already-resolved value, frozen; overriding a channel downstream doesn't force a recompute (verified in Chromium: the hover silently no-ops) | redeclare `--color-x: oklch(var(--x-l) var(--x-c) var(--x-h))` in the same rule that applies the channel override (e.g. on `.button` itself), see `color.md` |
| `:root { color-scheme: dark }` added preemptively before every surface has explicit color tokens | switches on the browser's UA dark defaults (canvas, form controls, unstyled text) anywhere `color`/`background` isn't explicit yet — breaks contrast silently | only set `color-scheme` once the real theme implementation covers every surface it affects |
| `light-dark(var(--color-x), var(--color-x-dark))` for a color that only gets darker | duplicates the whole color instead of moving a channel | redefine `--x-l-base` (and `-c-base` if needed) under `prefers-color-scheme: dark` |
| `--brand-l: 65%;` mixed with `--brand-c: 0.18;` | inconsistent units make `calc()` between channels error-prone | keep L and C unitless (`0.65`), H a bare degree number |
| `@media (max-width: 768px)` for an isolated component | media query where a container query belongs | `container-type: inline-size;` + `@container (width <= 48rem)` |
| Reordering elements in the DOM to change the mobile layout | couples structure to presentation | redefine `grid-template-areas` at the breakpoint |
| `!important` | breaks the cascade | move up a layer (`@layer`) or increase real specificity |
| A spacing value eyeballed from a mockup (`padding: 13px`) | assumes a measurement instead of confirming it | confirm the value with the requester, or use an existing token |
| A `-step-hover` delta invented on the spot ("looks about right") | still an unconfirmed value, same violation as any other assumed number | confirm it with the requester like any measurement |
| `.button {…}` followed by `.button:focus-visible {…}` as sibling rules | repeats the parent selector; the state no longer reads as part of the component | nest it: `.button { … &:focus-visible { … } }` |
| A `-base` channel block (`--x-l-base`/`-c-base`/`-h-base`) with no hex reference comment | bare OKLCH channels are unreadable — no way to recognize which color it is without resolving them | open the block with `/* brand — #b93f28 */` (see `color.md`) |
| `&--primary` written bare (no leading `.`) inside a nested rule | not Sass — CSS nesting parses the identifier after `&` as a type selector, not a class suffix; compiles to a broken selector that matches nothing (esbuild: `Cannot use type selector "--primary" directly after nesting selector "&"`) | a CUBE exception on `[data-variant='primary']` (nests cleanly, no suffix to break) — or, if the project already uses BEM/SMACSS, `&.button--primary`/`&.is-active` (full class, verified same DOM node) |
| Nesting more than 2 levels with `&` | excessive nesting, hard to read | concatenate an exception's state at level 2 (`&[data-variant='primary']:hover`); if it still needs more, flatten into its own block at the corresponding CUBE layer |
| `flex` with percentage widths + a media query for a uniform gallery | reimplements what grid does natively | `repeat(auto-fit, minmax(min(100%, <min>), 1fr))` |
| `margin-inline-start: auto` / a spacer element to push an item | margin hack instead of real alignment | `justify-content` / `align-items` on the container |
| `margin` between siblings to space them out | couples spacing to sibling order | `gap` on the flex/grid container |
| `padding-block-start: 56.25%` (padding-top proportion hack) | fragile, unitless-looking magic number | `aspect-ratio: 16 / 9` |
| fixed `height`/`block-size` to hold a media element's proportion | breaks at other sizes | `aspect-ratio` + `object-fit` |
| `grid-template-areas` forced onto a collection of N equal items | areas are for named/heterogeneous zones, not a repeated set | `repeat(auto-fit or auto-fill, minmax(...))` |
