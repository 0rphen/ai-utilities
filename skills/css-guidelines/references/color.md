# CSS Guidelines — Channel-Based Color System (L/C/H)

Read this whenever a task touches color: a new palette, a theme, dark mode, or any interactive state (hover/active/disabled/error/success).

## Why Atomic `oklch()` Isn't Enough

`--color-accent: oklch(65% 0.18 250)` is a single opaque value. To make a hover you either rewrite the whole triplet or reach for `color-mix()`, which blends all three channels at once in ways that are hard to predict and impossible to target — "a bit brighter, same hue" isn't expressible. The fix: **store L, C, and H as independent custom properties**, and compose the color from them. A state then means overriding the one channel that actually changes.

---

## Anatomy of a Color Token

```css
@layer tokens {
  :root {
    /* 1. base channels — the only raw numbers in the system, confirmed with the requester */
    /* brand — #4d8fff */
    --brand-l-base: 0.65;
    --brand-c-base: 0.18;
    --brand-h-base: 250;

    /* 2. live channels — what components override */
    --brand-l: var(--brand-l-base);
    --brand-c: var(--brand-c-base);
    --brand-h: var(--brand-h-base);
    --brand-a: 1;

    /* 3. composed color — resolved per element */
    --color-brand: oklch(
      var(--brand-l) var(--brand-c) var(--brand-h) / var(--brand-a)
    );
  }
}
```

L and C are unitless numbers (`0.65`, not `65%`), H is a bare degree number — this keeps every `calc()` a plain arithmetic expression with no unit juggling.

Every `-base` channel group opens with a hex reference comment, as shown above. It's not a consumable value and never becomes a custom property — it's a reading anchor so the color is recognizable without resolving OKLCH channels mentally. The hex comes from converting the actual source color (a prior palette, a mockup swatch, a value confirmed with the requester), never invented. The `-base` channels stay the only source of truth — if a `-base` changes, update the comment in the same edit or it goes stale.

---

## The Base/Live Split — the One Hard Rule

`--brand-l: calc(var(--brand-l) + 0.06)` is self-referential. A custom property can't be defined in terms of its own current value across a cascade recompute — the browser detects the cycle, the property becomes *guaranteed-invalid*, and the whole `--color-brand` composition breaks silently: no console error, just a wrong or transparent color. **Every delta is computed from `-base`, never from the live channel.**

### Why the composed color must be redeclared at the block

A custom property's computed value is fixed at the point in the cascade where that property was *last specified*. If `--color-brand` is declared only once, at `:root`, every descendant simply inherits that already-resolved value — frozen with whatever `--brand-l` was at `:root` at the time. Overriding `--brand-l` inside `.button:hover` does **not** make `--color-brand` recompute there, because `--color-brand` itself was never specified on `.button` — only inherited. The hover silently no-ops. Verified in real Chromium — this breaks silently otherwise.

The fix — redeclare the composed color in the same rule that applies the channel override:

```css
@layer block {
  .button {
    --color-brand: oklch(var(--brand-l) var(--brand-c) var(--brand-h) / var(--brand-a));
    background: var(--color-brand);
  }
  .button:hover {
    --brand-l: clamp(0, calc(var(--brand-l-base) + var(--l-step-hover)), 1);
  }
}
```

Now `--color-brand` is specified on `.button` itself, so its computed value is recalculated for that element (and its `:hover`/`:active` states) using whatever `--brand-l` resolves to right there. **Pitfall to recognize**: if a hover/active/disabled state changes a channel but the element's background/color visibly doesn't move at all, suspect exactly this — the composed `--color-*` token is declared only at `:root` (or some ancestor) and the component is consuming it without redeclaring it locally.

---

## States as Shared Deltas

Deltas are tokens, not literals repeated at every component that needs a hover. One number, tuned once, applied everywhere:

```css
@layer tokens {
  :root {
    --l-step-hover: 0.06;
    --l-step-active: -0.06;
    --c-step-muted: -0.08; /* desaturate without touching lightness or hue */
    --a-step-disabled: -0.6;
  }
}

@layer block {
  .button {
    /* redeclared here, not just at :root — see "the base/live split" above */
    --color-brand: oklch(var(--brand-l) var(--brand-c) var(--brand-h) / var(--brand-a));
    background: var(--color-brand);
  }
  .button:hover {
    --brand-l: clamp(0, calc(var(--brand-l-base) + var(--l-step-hover)), 1);
  }
  .button:active {
    --brand-l: clamp(0, calc(var(--brand-l-base) + var(--l-step-active)), 1);
  }
  .button:disabled {
    --brand-c: max(0, calc(var(--brand-c-base) + var(--c-step-muted)));
    --brand-a: calc(1 + var(--a-step-disabled));
  }
}
```

If the hover across the whole product needs to feel stronger, `--l-step-hover` is the single place to change — every component that follows this pattern moves together. Confirm deltas with the requester like any other value; a reasonable question shape is "how much lighter should hover feel — subtle / noticeable / strong" mapped to concrete options (`0.04` / `0.06` / `0.1`).

---

## Dark Mode = Redefine Channels, Not Colors

The default resolution: dark mode changes `-base` channels (mainly L, often C settles a bit lower too), H stays put because it's the same hue family in both themes.

```css
@layer tokens {
  :root {
    /* surface — #f4f9ff */
    --surface-l-base: 0.98;
    --surface-c-base: 0.01;
    --surface-h-base: 250;
  }

  @media (prefers-color-scheme: dark) {
    :root {
      /* surface dark — #0f171f */
      --surface-l-base: 0.2;
      --surface-c-base: 0.02;
    }
  }
}
```

If the project toggles theme via a class/attribute instead of (or in addition to) `prefers-color-scheme`, redefine the same `-base` custom properties under that selector (e.g. `:root[data-theme="dark"]`) — the mechanism for *how* dark mode activates doesn't change *what* gets redefined.

`light-dark()` is the documented exception, not the default — reach for it only when light and dark genuinely need a different hue, not just a different lightness of the same color (e.g. a brand mark that's warm-toned in light mode and cool-toned in dark mode).

Don't set `color-scheme` speculatively: `:root { color-scheme: dark }` switches on the browser's UA-default dark styling for any element without explicit `color`/`background` yet, breaking contrast silently on whatever hasn't been styled. Only set it once the real implementation covers every surface it touches.

---

## Complements

- **`@property`** — register a channel for type-checking, an `initial-value`, and interpolable transitions:

  ```css
  @property --brand-l {
    syntax: "<number>";
    inherits: true;
    initial-value: 0.65;
  }
  ```

  With this, `transition: --brand-l 150ms ease` animates the hover smoothly instead of snapping — plain custom properties can't be transitioned.

- **Range guards** — a delta must not push a channel out of gamut. Clamp L into `[0, 1]` with `clamp(0, …, 1)`, floor C at `0` with `max(0, …)`. Skipping this is how a "subtle" hover delta turns into a broken color on an already-bright base.

- **`color-mix(in oklch, …)`** is still correct for mixing two distinct colors — an overlay, a tint of the brand color over a surface — but not for deriving a state of the *same* color. If the goal is "this button, a bit brighter," that's a channel override, not a mix.

- **Relative color syntax** (`oklch(from var(--color-brand) calc(l + 0.06) c h)`) is a valid one-off for deriving from a color that isn't part of the token system — e.g. a color arriving from data at runtime. It's not a substitute for channel tokens on system colors.

- **Contrast** — L in OKLCH tracks perceptual lightness reasonably well, making it a cheap accessibility check when moving channels. Keep a documented minimum separation between a text token's `-l-base` and its surface token's `-l-base` (confirm the exact minimum with the requester per project), and re-check it whenever a state delta pushes L close to that boundary.

---

## Detecting an Existing Color System

Before introducing this system into a project, check whether existing color tokens already split into independent L/C/H channels or are atomic. If channels already exist, adopt that project's naming. If colors are atomic, do not refactor the whole project to the channel system — apply channels only to what's being created or touched, and report the inconsistency instead of silently fixing it everywhere.

## External Resources

- [oklch.com](https://oklch.com) — pick and convert OKLCH colors
- [MDN: `oklch()`](https://developer.mozilla.org/en-US/docs/Web/CSS/color_value/oklch)
- [MDN: `@property`](https://developer.mozilla.org/en-US/docs/Web/CSS/@property)
