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

## Internal Custom Properties (`--_*`)

An internal custom property is a block-scoped alias, declared at the top of the block, that stands between a public token and the declarations that use it. The leading `_` is a naming convention, not an engine-enforced boundary: CSS gives custom properties no visibility control, so `--_*` still inherits into every descendant and is still assignable by any selector that matches the block. The underscore signals "this is for this block's own use" — the barrier is authoring discipline (see the reassignment rule below), not something the cascade guarantees. For an engine-backed version of that boundary, see the optional `@property` hardening at the end of this section.

**When**: only for an axis something actually moves — a state, a `&[data-*]` exception, or a documented external reconfiguration (a theme, a host site that reskins the block). Reuse alone is not a reason: a block used on every page still consumes the token directly on every axis that never changes, because an internal with nothing to vary is dead indirection.

**How**: name it by role, not by the token it currently points to (`--_bg`, `--_radius`, `--_pad-block` — never `--_color-brand`, `--_radius-m`). One internal per configurable axis. Every declaration on that axis consumes the internal, never the public token in parallel. A state or exception reassigns the internal instead of repeating the declaration.

<!-- ✅ -->
```css
@layer block {
  /* --_radius is internal because [data-variant='flat'] moves it; padding never varies */
  .card {
    --_radius: var(--radius-m);

    container-type: inline-size;
    border-radius: var(--_radius);
    padding-block: var(--space-m);
    padding-inline: var(--space-s);

    &[data-variant='flat'] {
      --_radius: var(--radius-s);
    }
  }
}
```

<!-- ❌ never — the internal is declared but never consumed; border-radius still reads the public token, so the extension point does nothing -->
```css
@layer block {
  .card {
    --_radius: var(--radius-m);
    border-radius: var(--radius-m);
  }
}
```

Reassignment stays inside the block's own nesting — a state, or `&[data-variant='…']`. Never from a parent or a sibling block (`.sidebar .card { --_radius: … }`): that both breaks the block's encapsulation and crosses two classes in one selector, which the nesting rule already forbids. See [color.md](color.md) for internal customs applied to a state's color axis, and [cube.md](cube.md) for the block/exception placement this pattern lives in.

### Optional hardening with `@property`

The `_` convention alone doesn't stop inheritance or outside reassignment — CSS has no visibility model for custom properties. Registering an internal with `@property` and `inherits: false` closes the inheritance leak for real: the value stops propagating into descendants, so `.sidebar { --_radius: var(--radius-s); }` no longer bleeds into a `.card` nested inside it.

```css
@property --_radius {
  syntax: '*';
  inherits: false;
}
```

`syntax: '*'` is the only syntax that lets `initial-value` be omitted; the `@property` rule sits at the stylesheet's top level, not nested inside the block. This is optional, not a rule the other sections of this skill enforce — reach for it only when a block is reused inside other blocks that also touch `--_*`-shaped names and the inheritance leak is a real, observed risk, not by default on every internal.

Two things it does **not** give: a selector that matches the block itself still reassigns the internal (`.sidebar .card { --_radius: … }` still works — there's no privacy outside Shadow DOM), and `inherits: false` breaks any state or exception that reads `var(--_radius)` from a **descendant** element rather than the block's own selector or its `&`-nested states — those are unaffected since they're the same element, but a child element relying on inherited `--_radius` will see the registered initial value instead. Confirm nothing outside the block's own selector depends on inheriting the internal before registering it.

See [MDN: `@property`](https://developer.mozilla.org/en-US/docs/Web/CSS/@property).

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
