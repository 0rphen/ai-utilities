# CSS Guidelines — Color System (Base Tokens + Relative Color)

Read this whenever a task touches color: a new palette, a theme, dark mode, or any interactive state (hover/active/disabled/error/success).

## One Base Token per Color

A color is one custom property holding one complete value — `--color-brand: oklch(0.6 0.2 260)`. Its channels are never stored as separate custom properties: relative color syntax reads them straight off the token (`oklch(from var(--color-brand) l c h)`), so "a bit brighter, same hue" is expressible without splitting anything up front. Every state derives from that one token, computed right where it's used, by moving only the channel that actually changes.

---

## Anatomy of a Color Token

```css
@layer tokens {
  :root {
    --color-brand: oklch(0.6 0.2 260);
    --color-surface: oklch(0.98 0.01 250);
  }
}
```

`oklch()` is the preferred format, not a mandatory one — any valid CSS color works as a token's value, and the rest of this system behaves identically on top of it:

```css
@layer tokens {
  :root {
    --color-brand: #2e79f5; /* same color as above — equally valid as a base token */
  }
}
```

Why `oklch()` is preferred: lightness, chroma, and hue are readable on the token itself, so contrast and consistency between tokens can be checked by eye, and it reaches colors outside the sRGB gamut that hex/`rgb()`/`hsl()` can't express. When a color arrives in another format (a brand guide, a mockup swatch) and there's no reason to convert it, keep it as given.

Whatever the format, a color literal only ever appears as the value of a token in the tokens layer. Blocks, states, and exceptions consume the token — never a literal of their own.

---

## States via Relative Color Syntax

A state derives from the base token at the point of use — `oklch(from var(--color-brand) calc(l + var(--l-step-hover)) c h)`. The `from` keyword is required: it opens a scope where `l`, `c`, `h`, and `alpha` are the source color's own channels, available as plain numbers to `calc()`. Omitting `from` is invalid syntax — the declaration is dropped silently, no console error.

The source's format doesn't matter: `oklch(from …)` converts it to OKLCH first, so `l` and `c` are always unitless numbers (`l` in `[0, 1]`) and `h` a bare degree number, whether the token was written as `oklch()`, hex, or anything else. The same deltas apply to every token.

Because the state is computed directly in the declaration that uses it, there's no custom property to recompute and nothing frozen at `:root` — `--color-brand` is read once, live, wherever `oklch(from var(--color-brand) …)` appears.

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
    --_bg: var(--color-brand);
    background-color: var(--_bg);
    transition: background-color 150ms ease;

    &:hover {
      --_bg: oklch(from var(--color-brand) clamp(0, calc(l + var(--l-step-hover)), 1) c h);
    }
    &:active {
      --_bg: oklch(from var(--color-brand) clamp(0, calc(l + var(--l-step-active)), 1) c h);
    }
    &:disabled {
      --_bg: oklch(from var(--color-brand) l max(0, calc(c + var(--c-step-muted))) h / calc(alpha + var(--a-step-disabled)));
    }
  }
}
```

Three states move `.button`'s background, so that axis — and only that axis — is exposed as `--_bg`: `background-color` is declared once, and every state reassigns `--_bg` instead of repeating the property. The transition still fires normally, since it's watching `background-color` regardless of which value feeds it. See `tokens.md` for when this pattern is warranted versus a plain one-off declaration.

If the hover across the whole product needs to feel stronger, `--l-step-hover` is the single place to change — every component that follows this pattern moves together. Confirm deltas with the requester like any other value; a reasonable question shape is "how much lighter should hover feel — subtle / noticeable / strong" mapped to concrete options (`0.04` / `0.06` / `0.1`).

---

## Dark Mode = Redeclare the Token

The default resolution: dark mode redeclares the same token with its dark value. Nothing else moves — every state derives from the token live, so hover/active/disabled follow the new value without being touched.

```css
@layer tokens {
  :root {
    --color-surface: oklch(0.98 0.01 250);
  }

  @media (prefers-color-scheme: dark) {
    :root {
      --color-surface: oklch(0.2 0.02 250);
    }
  }
}
```

If the project toggles theme via a class/attribute instead of (or in addition to) `prefers-color-scheme`, redeclare the same tokens under that selector (e.g. `:root[data-theme="dark"]`) — the mechanism for *how* dark mode activates doesn't change *what* gets redeclared.

`light-dark()` is the documented exception, not the default — it only resolves once `color-scheme` is declared, so reach for it only in a project that already sets `color-scheme` deliberately and covers every surface it affects.

Don't set `color-scheme` speculatively: `:root { color-scheme: dark }` switches on the browser's UA-default dark styling for any element without explicit `color`/`background` yet, breaking contrast silently on whatever hasn't been styled. Only set it once the real implementation covers every surface it touches.

---

## Complements

- **Transitions come for free** — since a state is a value on `background-color`/`color` itself (`oklch(from var(--color-brand) …)`), a plain `transition: background-color 150ms ease` on the block animates the hover/active swap smoothly. No `@property` registration needed for this.

- **Range guards** — a delta must not push a channel out of gamut. Clamp L into `[0, 1]` with `clamp(0, …, 1)`, floor C at `0` with `max(0, …)`. Skipping this is how a "subtle" hover delta turns into a broken color on an already-bright base.

- **`color-mix(in oklch, …)`** is still correct for mixing two distinct colors — an overlay, a tint of the brand color over a surface — but not for deriving a state of the *same* color. If the goal is "this button, a bit brighter," that's a relative-color derivation, not a mix.

- **Relative color syntax is the standard state mechanism** on system colors (`oklch(from var(--color-brand) calc(l + var(--l-step-hover)) c h)`), not just a one-off — but it also covers a color that isn't part of the token system, e.g. one arriving from data at runtime, the same way.

- **Contrast** — L in OKLCH tracks perceptual lightness reasonably well, making it a cheap accessibility check. Keep a documented minimum separation between a text token's L and its surface token's L (confirm the exact minimum with the requester per project), and re-check it whenever a state delta pushes L close to that boundary. With tokens written in `oklch()` the L is readable directly; for a token in another format, convert it to check.

---

## Detecting an Existing Color System

Before introducing this system into a project, check how its existing color tokens are defined. If they're already one token per color — in any format — use them as they are; don't convert them to `oklch()` unasked. If the project stores L/C/H as separate custom properties and composes colors from them, adopt that convention for what's being created or touched instead of mixing in a second one, and report the inconsistency rather than silently refactoring it everywhere.

## External Resources

- [oklch.com](https://oklch.com) — pick and convert OKLCH colors
- [MDN: `oklch()`](https://developer.mozilla.org/en-US/docs/Web/CSS/color_value/oklch)
- [MDN: relative colors](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_colors/Relative_colors)
