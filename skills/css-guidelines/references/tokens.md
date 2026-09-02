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

## Private Custom Properties (`--_*`)

A private custom property is a block-scoped alias, declared at the top of the block, that stands between a public token and the declarations that use it. The leading `_` means "private to this block — nothing outside reads or sets it directly."

**When**: only if the block is reused across different sites with different configurations, or has states/variants that move one of its values. A single-use block with fixed values consumes the token directly — a private with nothing to vary is dead indirection.

**How**: name it by role, not by the token it currently points to (`--_bg`, `--_radius`, `--_pad-block` — never `--_color-brand`, `--_radius-m`). One private per configurable axis. Every declaration on that axis consumes the private, never the public token in parallel. A state or exception reassigns the private instead of repeating the declaration.

<!-- ✅ -->
```css
@layer block {
  /* private customs because .card is reused across sites with different configurations */
  .card {
    --_radius: var(--radius-m);
    --_pad-block: var(--space-m);
    --_pad-inline: var(--space-s);

    container-type: inline-size;
    border-radius: var(--_radius);
    padding-block: var(--_pad-block);
    padding-inline: var(--_pad-inline);

    &[data-variant='flat'] {
      --_radius: var(--radius-s);
    }
  }
}
```

<!-- ❌ never — the private is declared but never consumed; border-radius still reads the public token, so the extension point does nothing -->
```css
@layer block {
  .card {
    --_radius: var(--radius-m);
    border-radius: var(--radius-m);
  }
}
```

Reassignment stays inside the block's own nesting — a state, or `&[data-variant='…']`. Never from a parent or a sibling block (`.sidebar .card { --_radius: … }`): that both breaks the block's encapsulation and crosses two classes in one selector, which the nesting rule already forbids. See [color.md](color.md) for private customs applied to a state's color axis, and [cube.md](cube.md) for the block/exception placement this pattern lives in.

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
