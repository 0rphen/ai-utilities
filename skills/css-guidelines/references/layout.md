# CSS Guidelines — Layout & Responsive Patterns

Choosing the right layout primitive — and the right kind of responsiveness — is a placement decision, not a matter of taste. This file covers grid, subgrid, flex, alignment, proportions, and how media, container size, and container style queries divide the work.

## Named Layout with Grid + Container Queries

A heterogeneous set of zones (card, page, dashboard) always uses `grid-template-areas` with semantic names. Responsive means **redefining areas at a breakpoint**, never moving nodes in the DOM or reaching for a fixed-width media query by default:

```css
@layer block {
  .card {
    container-type: inline-size;
    container-name: card;

    display: grid;
    grid-template-areas:
      "media"
      "body"
      "actions";
    gap: var(--space-s);
  }

  .card__media { grid-area: media; }
  .card__body { grid-area: body; }
  .card__actions { grid-area: actions; }

  @container card (width >= 28rem) {
    .card {
      grid-template-columns: minmax(10rem, 1fr) 2fr;
      grid-template-areas:
        "media body"
        "media actions";
    }
  }
}
```

---

## Container Style Queries for Context

A size query answers "how much room do I have?". A style query answers "what context did my container declare?" — density, surface, emphasis, or whatever modes a project actually needs. It's how a container configures the blocks inside it without touching their markup and without a descendant selector reaching into them. It complements `[data-*]` exceptions rather than replacing them: an exception is a block's **own** variant, set on the block itself; a style query reacts to context set by an **ancestor**.

The pattern, independent of which context properties a project defines:

- **Context property** — a public custom property named by role, no `_` prefix, holding keyword values. Its default lives on `:root` in the tokens layer; a `.l-*` container overrides it in the composition layer.
- **Consumer** — the block nests `@container style(--x: value)` inside its own rule and only reassigns its `--_*` internals there; every final declaration stays single, consuming the internal.

`--density` below is illustrative — name and values come from the project, never from this example:

<!-- ✅ -->
```css
@layer tokens {
  :root {
    --density: comfortable;
  }
}

@layer composition {
  .l-grid {
    &[data-density='compact'] {
      --density: compact;
    }
  }
}

@layer block {
  .card {
    --_pad-block: var(--space-m);
    --_gap: var(--space-s);

    padding-block: var(--_pad-block);
    gap: var(--_gap);

    @container style(--density: compact) {
      --_pad-block: var(--space-xs);
      --_gap: var(--space-2xs);
    }
  }
}
```

<!-- ❌ never — literals instead of tokens, final properties redeclared instead of internals reassigned -->
```css
@container style(--density: compact) {
  .card-bad {
    padding: 0.75rem;
    gap: 0.5rem;
  }
}
```

<!-- ❌ never — a descendant selector reaching into the block from its container -->
```css
.l-grid-compact .card-bad {
  --_pad-block: var(--space-xs);
}
```

Gotchas that fail silently:

- **The query evaluates an ancestor, never the element itself** — a block can't query a context property it sets on its own selector. The context belongs on a container.
- **No `container-type` needed** — every element is a style container; `container-type` is only for size queries. Name the container (`@container grid style(--x: y)`) only when a nearer ancestor would shadow the one that matters.
- **Context inherits to any depth** — custom properties inherit, so every block inside the container sees the value, and a nested container that redeclares it wins for its own subtree.
- **Values match as exact token strings** — `compact` and `"compact"` differ. Optionally register the property with `@property` and a keyword syntax (`syntax: 'compact | comfortable'`) so invalid values are rejected at parse time.
- **Not a dark-mode mechanism** — dark mode still redeclares color tokens (see `color.md`); a surface-style context reassigns a block's `--_bg` between existing color tokens, never introduces a literal.

---

## Subgrid for Nested Alignment

When a child must line up with its parent's tracks instead of redeclaring its own columns:

```css
@layer composition {
  .l-page {
    display: grid;
    grid-template-columns: repeat(12, 1fr);
    gap: var(--space-m);
  }

  .l-page__section {
    grid-column: 1 / -1;
    display: grid;
    grid-template-columns: subgrid;
  }
}
```

---

## Flex for One-Dimensional Arrangements

Rows, lists, action groups, a single stacked column. Responsiveness comes from `flex-wrap`, never a breakpoint:

```css
@layer composition {
  .l-cluster {
    display: flex;
    flex-wrap: wrap;
    gap: var(--space-2xs);
    align-items: center;
  }
}
```

---

## Homogeneous Collections with `auto-fit` / `auto-fill`

A repeated set of equal items (cards, gallery tiles) has no zones to name — the tracks are implicit, so `grid-template-areas` doesn't apply here. `auto-fit` collapses empty tracks so items stretch to fill the row; `auto-fill` keeps them, holding item width stable as the container grows:

```css
@layer composition {
  .l-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(min(100%, 18rem), 1fr));
    gap: var(--space-m);
  }
}
```

`min(100%, 18rem)` keeps the minimum from overflowing a container narrower than 18rem.

---

## Alignment via `justify-content` / `align-items`

<!-- ✅ -->
```css
.l-toolbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: var(--space-s);
}
```

<!-- ❌ never — margin hack instead of real alignment -->
```css
.l-toolbar-bad .button:last-child {
  margin-inline-start: auto;
}
```

Separate items with `gap`, never margins between siblings.

---

## `aspect-ratio` for Proportions

<!-- ✅ -->
```css
.thumb {
  aspect-ratio: 16 / 9;
  inline-size: 100%;
}
.thumb img {
  inline-size: 100%;
  block-size: 100%;
  object-fit: cover;
}
```

<!-- ❌ never — the padding-top proportion hack -->
```css
.thumb-bad {
  padding-block-start: 56.25%;
}
```

---

## Media Queries — Range Syntax, Global Concerns Only

Container queries are the default. Media queries are reserved for genuinely global (viewport, not component) concerns, and always written in range syntax:

```css
@layer composition {
  .l-page {
    padding-inline: var(--space-s);
  }

  @media (48rem <= width <= 64rem) {
    .l-page {
      padding-inline: var(--space-l);
    }
  }
}
```

Ask "does this need to know about the viewport, or just its own available space?" — a card, an about-us block, or a sidebar reflowing on its own is a component concern even when it spans the full page width.

---

## Logical Properties, Always

<!-- ✅ -->
```css
.panel {
  padding-block: var(--space-m);
  padding-inline: var(--space-s);
  margin-block-end: var(--space-l);
  border-inline-start: 0.125rem solid var(--color-accent);
  inset-block-start: 0;
}
```

<!-- ❌ never — physical properties -->
```css
.panel-bad {
  padding-top: 1.5rem;
  padding-left: 1rem;
  margin-bottom: 2rem;
  border-left: 2px solid red;
}
```
