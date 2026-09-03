# CSS Guidelines — CUBE CSS Structure & Naming

CUBE classifies a rule by the **role it plays in the cascade** — not by how complex the component it belongs to looks. Reach for this whenever a rule needs a home: is it layout skeleton, the component itself, a single-property helper, or a variation of an existing component?

## The Cascade Layers

```css
@layer reset, tokens, composition, block, utility, exception;
```

`utility` sits after `block` so a utility class wins over a block's own rule without reaching for `!important` — that's the point of having utilities at all. `exception` is last because it is, by design, the most specific override a block can have.

---

## Composition (`.l-*`)

Layout skeleton, agnostic of appearance. No color, no typography, no decoration — only arrangement (`display`, `gap`, `grid-template-areas`, `flex-direction`). Reusable across unrelated blocks.

```css
@layer composition {
  .l-cluster {
    display: flex;
    flex-wrap: wrap;
    align-items: center;
    gap: var(--space-s);
  }

  .l-sidebar {
    display: grid;
    grid-template-columns: minmax(16rem, 1fr) 3fr;
    gap: var(--space-l);
  }
}
```

---

## Block

The component itself — its appearance and behavior. Semantic name, **no prefix**. This is the layer people mean when they say "a component."

```css
@layer block {
  .card {
    container-type: inline-size;
    border-radius: var(--radius-m);
    padding-block: var(--space-m);
    padding-inline: var(--space-s);
  }
}
```

Every axis here has a fixed value, so each declaration reads its token directly. Only an axis something actually moves — a state, or one of the exceptions below — is exposed as an internal custom property (`--_*`); see the Exception section and `tokens.md` for the criterion.

---

## Utility (`.u-*`)

A single property (or a tightly-coupled group) fed by a token, with no state of its own. Last-mile overrides, applied directly in markup.

```css
@layer utility {
  .u-text-center {
    text-align: center;
  }

  .u-visually-hidden {
    position: absolute;
    inline-size: 1px;
    block-size: 1px;
    overflow: hidden;
    clip-path: inset(50%);
    white-space: nowrap;
  }
}
```

---

## Exception (`[data-*]`)

A variation of an existing block — never a standalone class, never its own layer per component. Expressed with `[data-variant]` / `[data-state]`, nested inside the block it belongs to:

```css
@layer block {
  .card {
    --_radius: var(--radius-m);
    border-radius: var(--_radius);

    &[data-variant='ghost'] {
      background: transparent;
      border: var(--border-hairline) solid var(--color-neutral);
    }

    &[data-variant='flat'] {
      --_radius: var(--radius-s);
    }

    &[data-state='loading'] {
      cursor: progress;
      opacity: 0.6;
    }
  }
}
```

When the exception's job is to move a value the block already exposes as an internal (like `--_radius` above), it reassigns that internal rather than redeclaring the final property — the block keeps a single declaration for that property. `[data-variant='ghost']` still declares its own properties directly because `background`/`border` aren't internal axes of `.card` in this example.

---

## Folder Structure

A reasonable default for a project with no prior convention:

```
styles/
  tokens/         → @layer tokens (colors, spacing, typography, radii)
  composition/    → @layer composition — .l-stack, .l-cluster, .l-sidebar, .l-switcher
  blocks/         → @layer block       — .card, .button, .field, .calendar
  utilities/      → @layer utility     — .u-*
                    @layer exception   — no folder of its own: lives nested inside its block
```

---

## Adopt Before Imposing

If the project already has a naming convention (BEM, SMACSS, Atomic Design, utility-first, CSS Modules, free semantic naming), an `@layer` order, tokens, or a folder structure, **use those** — never mix a second convention into the same tree. These rules apply to what's being created or touched, never as a silent wholesale refactor of the rest of the project; if something existing conflicts with these rules, report the inconsistency instead of fixing it unasked.

---

## Decision Table

| Question | Answer | Layer |
| --- | --- | --- |
| Is it pure arrangement, no color/typography/decoration? | Yes | Composition |
| Is it the component's own appearance/behavior? | Yes | Block |
| Is it a single property (or tightly-coupled group) applied directly in markup? | Yes | Utility |
| Is it a variation of a block that already exists? | Yes | Exception, nested inside that block |

## External Resources

- [CUBE CSS](https://cube.fyi)
- [MDN: `@layer`](https://developer.mozilla.org/en-US/docs/Web/CSS/@layer)
- [MDN: CSS nesting](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_nesting)
