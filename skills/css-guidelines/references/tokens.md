# CSS Guidelines — Tokens & Fluid Scales

All general values — spacing, radii, typography, durations — live as custom properties in a dedicated tokens layer, fed by a value confirmed with whoever requested the work. Never a made-up number.

## The Tokens Layer

Declare the full cascade once, with `tokens` right after `reset`:

```css
@layer reset, tokens, composition, block, utility, exception;

@layer tokens {
  :root {
    /* spacing — geometric scale, values confirmed with the requester */
    --space-3xs: 0.25rem;
    --space-2xs: 0.5rem;
    --space-xs: 0.75rem;
    --space-s: 1rem;
    --space-m: 1.5rem;
    --space-l: 2rem;
    --space-xl: 3rem;

    /* radii */
    --radius-s: 0.25rem;
    --radius-m: 0.5rem;
    --radius-full: 999rem;

    /* fluid typography */
    --font-size-body: clamp(1rem, 0.95rem + 0.25vw, 1.125rem);
    --font-size-heading: clamp(1.5rem, 1.2rem + 1.5vw, 2.5rem);
  }
}
```

Color tokens follow the same layer but need their own anatomy — see [color.md](color.md) for the L/C/H channel split.

---

## Fluid Sizing with `clamp()` / `min()` / `max()`

Use `clamp()` to let a value scale smoothly between a floor and a ceiling, and `min()`/`max()` to cap a value against another unit:

```css
.heading {
  font-size: var(--font-size-heading);
  max-inline-size: min(60ch, 90%);
}

.l-sidebar {
  inline-size: clamp(16rem, 25vw, 22rem);
}
```

---

## Where Numbers Come From

Every number written into CSS comes from an existing project token, or a value confirmed with whoever requested the work. Never a placeholder, never "a reasonable value for now," never a number eyeballed off a mockup. When a needed value is neither tokenized nor stated, ask for it before writing it.
