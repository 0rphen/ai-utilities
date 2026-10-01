# css-guidelines

A Claude Code skill with modern CSS/SCSS authoring rules: cascade layers (CUBE CSS), grid/container queries (size and style), logical properties, design tokens, and a token-based color system (OKLCH preferred) with predictable states and dark mode.

## Install

Clone (or copy) this repo into your skills directory:

```sh
git clone <this-repo-url> ~/.claude/skills/css-guidelines
```

Or add it as a plugin per your Claude Code setup. Once installed, Claude reads `SKILL.md` before writing, editing, or reviewing any CSS/SCSS or `<style>` block.

## Contents

- `SKILL.md` — the rules: cascade, placement, layout, responsiveness, units, tokens, color, nesting, accessibility.
- `references/tokens.md` — the tokens layer, internal custom properties (`--_*`) for a block's varying axes, and fluid scales.
- `references/layout.md` — grid, subgrid, flex, container size/style queries, media queries, logical properties.
- `references/cube.md` — CUBE CSS layers, naming, folder structure.
- `references/color.md` — base color tokens, relative-color states, dark mode.
- `references/antipatterns.md` — antipattern → replacement table, used in audit mode.
