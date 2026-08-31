# CSS Guidelines — Channel-Based Color System (L/C/H)

Read this whenever a task touches color: a new palette, a theme, dark mode, or any interactive state (hover/active/disabled/error/success).

## Why Atomic `oklch()` Isn't Enough

`--color-accent: oklch(65% 0.18 250)` is a single opaque value. To make a hover you either rewrite the whole triplet or reach for `color-mix()`, which blends all three channels at once in ways that are hard to predict and impossible to target — "a bit brighter, same hue" isn't expressible. The fix: **store L, C, and H as independent custom properties**, compose the color once from them, and derive every state from that composed color with relative color syntax. A state then means overriding the one channel that actually changes, computed right where it's used.

---

## Anatomy of a Color Token

```css
@layer tokens {
  :root {
    /* brand — #4d8fff */
    --brand-l: 0.65;
    --brand-c: 0.18;
    --brand-h: 250;
    --brand-a: 1;

    /* composed color — the only thing components consume */
    --color-brand: oklch(
      var(--brand-l) var(--brand-c) var(--brand-h) / var(--brand-a)
    );
  }
}
```

L and C are unitless numbers (`0.65`, not `65%`), H is a bare degree number — this keeps every `calc()` a plain arithmetic expression with no unit juggling.

Every channel group opens with a hex reference comment, as shown above. It's not a consumable value and never becomes a custom property — it's a reading anchor so the color is recognizable without resolving OKLCH channels mentally. The hex comes from converting the actual source color (a prior palette, a mockup swatch, a value confirmed with the requester), never invented. The channels stay the only source of truth — if one changes, update the comment in the same edit or it goes stale.

---

## States via Relative Color Syntax

A state derives from the composed color at the point of use — `oklch(from var(--color-brand) calc(l + var(--l-step-hover)) c h)`. The `from` keyword is required: it opens a scope where `l`, `c`, `h`, and `alpha` are the source color's own channels, available as plain numbers to `calc()`. Omitting `from` is invalid syntax — the declaration is dropped silently, no console error.

Because the state is computed directly in the declaration that uses it, there's no custom property to recompute and nothing frozen at `:root` — `--color-brand` is read once, live, wherever `oklch(from var(--color-brand) …)` appears. This is also why the old base/live channel split and the "redeclare the composed color at every block" rule are gone: there's only one channel set, and no cascade recomputation to work around.

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
    background-color: var(--color-brand);
    transition: background-color 150ms ease;

    &:hover {
      background-color: oklch(from var(--color-brand) clamp(0, calc(l + var(--l-step-hover)), 1) c h);
    }
    &:active {
      background-color: oklch(from var(--color-brand) clamp(0, calc(l + var(--l-step-active)), 1) c h);
    }
    &:disabled {
      background-color: oklch(from var(--color-brand) l max(0, calc(c + var(--c-step-muted))) h / calc(alpha + var(--a-step-disabled)));
    }
  }
}
```

If the hover across the whole product needs to feel stronger, `--l-step-hover` is the single place to change — every component that follows this pattern moves together. Confirm deltas with the requester like any other value; a reasonable question shape is "how much lighter should hover feel — subtle / noticeable / strong" mapped to concrete options (`0.04` / `0.06` / `0.1`).

---

## Dark Mode = Redefine Channels, Not Colors

The default resolution: dark mode changes the channels directly (mainly L, often C settles a bit lower too), H stays put because it's the same hue family in both themes.

```css
@layer tokens {
  :root {
    /* surface — #f4f9ff */
    --surface-l: 0.98;
    --surface-c: 0.01;
    --surface-h: 250;
  }

  @media (prefers-color-scheme: dark) {
    :root {
      /* surface dark — #0f171f */
      --surface-l: 0.2;
      --surface-c: 0.02;
    }
  }
}
```

If the project toggles theme via a class/attribute instead of (or in addition to) `prefers-color-scheme`, redefine the same channel custom properties under that selector (e.g. `:root[data-theme="dark"]`) — the mechanism for *how* dark mode activates doesn't change *what* gets redefined.

`light-dark()` is the documented exception, not the default — reach for it only when light and dark genuinely need a different hue, not just a different lightness of the same color (e.g. a brand mark that's warm-toned in light mode and cool-toned in dark mode).

Don't set `color-scheme` speculatively: `:root { color-scheme: dark }` switches on the browser's UA-default dark styling for any element without explicit `color`/`background` yet, breaking contrast silently on whatever hasn't been styled. Only set it once the real implementation covers every surface it touches.

---

## Complements

- **Transitions come for free** — since a state is a value on `background-color`/`color` itself (`oklch(from var(--color-brand) …)`), a plain `transition: background-color 150ms ease` on the block animates the hover/active swap smoothly. No `@property` registration needed for this.

- **Range guards** — a delta must not push a channel out of gamut. Clamp L into `[0, 1]` with `clamp(0, …, 1)`, floor C at `0` with `max(0, …)`. Skipping this is how a "subtle" hover delta turns into a broken color on an already-bright base.

- **`color-mix(in oklch, …)`** is still correct for mixing two distinct colors — an overlay, a tint of the brand color over a surface — but not for deriving a state of the *same* color. If the goal is "this button, a bit brighter," that's a relative-color derivation, not a mix.

- **Relative color syntax is the standard state mechanism** on system colors (`oklch(from var(--color-brand) calc(l + var(--l-step-hover)) c h)`), not just a one-off — but it also covers a color that isn't part of the token system, e.g. one arriving from data at runtime, the same way.

- **Contrast** — L in OKLCH tracks perceptual lightness reasonably well, making it a cheap accessibility check when moving channels. Keep a documented minimum separation between a text token's `-l` and its surface token's `-l` (confirm the exact minimum with the requester per project), and re-check it whenever a state delta pushes L close to that boundary.

---

## Detecting an Existing Color System

Before introducing this system into a project, check whether existing color tokens already split into independent L/C/H channels or are atomic. If channels already exist, adopt that project's naming. If colors are atomic, do not refactor the whole project to the channel system — apply channels only to what's being created or touched, and report the inconsistency instead of silently fixing it everywhere.

## External Resources

- [oklch.com](https://oklch.com) — pick and convert OKLCH colors
- [MDN: `oklch()`](https://developer.mozilla.org/en-US/docs/Web/CSS/color_value/oklch)
- [MDN: `@property`](https://developer.mozilla.org/en-US/docs/Web/CSS/@property)
