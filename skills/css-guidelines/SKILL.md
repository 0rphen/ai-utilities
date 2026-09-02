---
name: css-guidelines
description: Modern CSS/SCSS authoring rules — cascade layers, CUBE CSS placement, grid/container queries, logical properties, design tokens, and an OKLCH L/C/H channel color system. Use when writing, editing, reviewing or auditing any CSS, SCSS, or `<style>` block in a .vue/.svelte/.astro component, or making any styling decision (layout, spacing, units, breakpoints, tokens, color, states, dark mode).
allowed-tools: Read, Write, Edit, Glob, Grep
---

# Modern CSS Authoring Guidelines

A single source of truth for how CSS/SCSS gets written, in any project or context — from scratch, as a targeted fix, or under audit. Not a framework, not a reset — a body of rules to write by or to check against, covering cascade structure, layout, responsiveness, units, tokens, color, nesting, and accessibility.

## Overview

Provides concrete rules for structuring the cascade, placing every rule in the layer it belongs to, and making layout, color, and spacing decisions consistently across a codebase. Covers cascade layers (CUBE CSS), container/media queries, logical properties, fluid tokens, and a channel-based OKLCH color system with predictable states and dark mode.

These rules apply to what's being created or touched — never as a silent wholesale rewrite of a project's existing conventions. If a project already has its own naming convention, layer order, or token system, adopt it instead of imposing a second one; report a conflict rather than fixing it unasked.

## Instructions

1. **Adopt before imposing**: check for an existing naming convention (BEM, SMACSS, Atomic Design, utility-first, CSS Modules, free semantic naming), `@layer` order, tokens, or folder structure, and use those rather than mixing in a second system.
2. **Declare the cascade once**: `@layer reset, tokens, composition, block, utility, exception;` — `utility` sits after `block` so a utility class wins without `!important`; `exception` is last because it's the most specific override a block can have by design. `!important` is forbidden — move up a layer or increase real specificity instead.
3. **Place every rule by CUBE role**, not by how complex the component looks — layout skeleton, the component's own appearance, a single-property helper, or a variation of an existing block. See `references/cube.md`.
4. **Choose the right layout tool**: flex for one-dimensional arrangements (responsive via `flex-wrap`, never a breakpoint); grid + `grid-template-areas` for named/heterogeneous zones, redefined at a breakpoint rather than reordering DOM nodes; `repeat(auto-fit/auto-fill, minmax(...))` for homogeneous collections; subgrid when a child must align to a parent's tracks. See `references/layout.md`.
5. **Prefer container queries**: `container-type` + `@container` by default; reserve media queries for genuinely global (viewport) concerns, always in range syntax (`@media (400px <= width <= 900px)`). A section that reflows on its own is a component concern even if it spans the full page.
6. **Use relative units only**: `px` is forbidden except for an explicitly justified 1px hairline. Reach for `ch`, `rem`, `em`, `%`, `fr`, `dvh`/`dvi`/`svh`/`dvw`/`dvb`/`svw`, `vw`/`vh`, and the `cq*` family.
7. **Use logical properties, always**: `padding-inline`/`padding-block`, `margin-inline`/`margin-block`, `inset-*`, `border-inline-*`/`border-block-*` — never physical `padding-left`, `margin-top`, `left`/`right`.
8. **Source general values from tokens**: border-radius, padding, margin, gap, font-size, duration all come from custom properties in a tokens layer, never repeated literals. Fluid scales use `clamp()`, caps use `min()`/`max()`. See `references/tokens.md`.
9. **Compose color from independent L/C/H channels** in `oklch()` — never an atomic literal, never `rgb()`/`hsl()`/hex/named colors (the only exceptions are `transparent` and `currentColor`). Dark mode redefines the channels directly; states derive from the composed color at the point of use with relative color syntax (`oklch(from var(--color-x) calc(l + var(--l-step-hover)) c h)`) and a shared delta token. See `references/color.md`.
10. **Expose only a block's varying axes through private custom properties**: declare `--_*` at the top of the block for an axis something actually moves — a state, a `&[data-*]` exception, or a documented external reconfiguration (theme/host) — name it by role (`--_bg`, `--_radius`, `--_pad-block`), make every declaration on that axis consume only the private, and reassign the private in the state instead of repeating the declaration. An axis with a fixed value consumes the token directly, however widely the block is reused. See `references/tokens.md`.
11. **Nest only what compounds onto the same selector**: pseudo-classes, pseudo-elements, at-rules that modify the block, and exceptions nest with `&`, max ~2 levels deep. Never write a bare `&--suffix` — it parses as a type selector, not a class suffix. Keep selectors crossing two distinct classes or DOM nodes flat.
12. **Meet the accessibility floor**: `:focus-visible` always visible, touch targets ≥ `2.75rem`, respect `prefers-reduced-motion`, and keep a documented minimum lightness separation between a text token and its surface token.
13. **Never invent a number**: every value comes from an existing project token or one confirmed with whoever requested the work — not a placeholder, not something eyeballed off a mockup.
14. **In review mode**, walk `references/antipatterns.md` and report each match as `file:line → antipattern → concrete replacement` without applying the fix — the requester decides.

## Best Practices

1. **Consistent token scale**: draw spacing, radii, and type sizes from one geometric scale instead of ad hoc values per component.
2. **Tokens over literals**: a repeated raw value anywhere outside the tokens layer is a smell — promote it.
3. **`gap` over margin hacks**: separate flex/grid children with `gap`, never margin between siblings or a spacer element.
4. **`aspect-ratio` over fixed heights**: hold media/thumbnail proportions with `aspect-ratio`, not a fixed `height` or the padding-top hack.
5. **Exceptions via `[data-*]`, nested**: a block variation is an attribute selector nested inside its block, never a standalone class or its own layer.
6. **Semantic HTML first**: styling decisions assume the right element is already chosen; CSS never compensates for the wrong one.

## Troubleshooting

### A hover/active/disabled state applies no color at all
The declaration is missing the `from` keyword — `oklch(var(--color-x) calc(l + …) c h)` is invalid syntax and gets dropped silently, no console error. **Fix**: `oklch(from var(--color-x) calc(l + var(--l-step-hover)) c h)`; `l`/`c`/`h`/`alpha` are only available as plain numbers inside a `from` scope.

### A nested modifier selector matches nothing
`&--primary` is not Sass string concatenation — CSS nesting parses the identifier right after `&` as a type selector, compiling to a broken rule. **Fix**: use a CUBE exception (`&[data-variant='primary']`), or, in a BEM/SMACSS project, the full class with its leading dot (`&.button--primary`), verified to sit on the same DOM node.

### Contrast breaks on an unstyled element after adding dark mode
`color-scheme` set preemptively (`:root { color-scheme: dark }`) switches on the browser's UA dark defaults for anything without an explicit `color`/`background` yet. **Fix**: only set `color-scheme` once every surface it affects has explicit color tokens.

## Constraints and Warnings

- **`!important` is forbidden** — move up a layer or raise real specificity.
- **`px` is forbidden** outside an explicitly justified 1px hairline.
- **No hex, `rgb()`, `hsl()`, or named colors** (`white`, `black`, `red`, …) — only `transparent` and `currentColor` are exempt, since neither names an actual color.
- **No physical box properties** — logical properties only.
- **Never reorder DOM nodes to change a responsive layout** — redefine `grid-template-areas` at the breakpoint instead.
- **`color-mix()` is not a state tool** — correct for blending two distinct colors, wrong for deriving a state of the same color (use relative color syntax with a channel delta).
- **`light-dark()` is the exception, not the default** — reach for it only when light/dark genuinely need a different hue, not just a different lightness.
- **No invented numbers, ever** — not a placeholder, not "reasonable for now."
- **A private (`--_*`) is declared only for an axis something moves** — reuse alone never justifies one; a fixed value consumes the token directly.
- **A declared private (`--_*`) is always consumed** — declaring `--_x` and still reading the public token in the declaration is dead indirection.
- **`--_*` is never reassigned from outside its block** — not from a parent selector, not from another block.

## References

- **[references/tokens.md](references/tokens.md)** — the tokens layer, spacing/radius/typography scales, private custom properties (`--_*`) for a block's varying axes, and fluid sizing with `clamp()`/`min()`/`max()`. Open before adding any general numeric value or exposing an axis a block's states actually move.
- **[references/layout.md](references/layout.md)** — grid-template-areas with container queries, subgrid, flex, homogeneous collections, alignment, `aspect-ratio`, range-syntax media queries, and logical properties. Open before laying out any component or page section.
- **[references/cube.md](references/cube.md)** — the four CUBE layers, naming, folder structure, and the placement decision table. Open whenever a new rule needs a home.
- **[references/color.md](references/color.md)** — the L/C/H channel color system: token anatomy, states via relative color syntax, dark mode, and the gotchas that break silently. Open before touching any palette, theme, dark mode, or interactive color state.
- **[references/antipatterns.md](references/antipatterns.md)** — the antipattern → replacement table used in review/audit mode.
