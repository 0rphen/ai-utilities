# ia-utilities

*[Leer en español](README.es.md)*

A collection of skills and utilities for [Claude Code](https://claude.ai/code).

## What's here

| Skill | Description |
| --- | --- |
| [`css-guidelines`](skills/css-guidelines/) | Modern CSS/SCSS authoring rules — cascade layers, CUBE CSS placement, grid/container queries, logical properties, design tokens, and an OKLCH L/C/H channel color system. |
| [`frontend-arch`](skills/frontend-arch/) | Framework-agnostic frontend architecture rules — layering and dependency direction, feature-first folder structure, atomic-design component decomposition, and server/client state placement. |

## Install

### A single skill

Clone this repo and copy (or symlink) the skill you want into your skills directory:

```sh
git clone <this-repo-url> ia-utilities
ln -s "$(pwd)/ia-utilities/skills/css-guidelines" ~/.claude/skills/css-guidelines
```

### All skills

```sh
git clone <this-repo-url> ia-utilities
for skill in ia-utilities/skills/*/; do
  ln -s "$(pwd)/$skill" ~/.claude/skills/"$(basename "$skill")"
done
```

Once installed, each skill auto-loads based on its `description` frontmatter — Claude Code decides when it's relevant, there's no need to invoke it by name.

## Repository layout

```
skills/
  <skill-name>/
    SKILL.md          # rules and frontmatter (name, description, allowed-tools) — source of truth
    references/*.md    # detail docs the skill links to on demand, not preloaded
    README.md          # human-facing install/contents notes for that skill
```

Each skill lives in its own directory under `skills/`. `SKILL.md` is what Claude Code reads to decide when and how to apply the skill; anything in `references/` is loaded only when `SKILL.md` points into it.

## Adding a skill

1. Create a new directory under `skills/<name>/`.
2. Add a `SKILL.md` with `name`, `description`, and `allowed-tools` frontmatter, followed by the rules.
3. Optionally add `references/*.md` for detail the skill can link into on demand, and a `README.md` for humans browsing the repo.
4. Add a row to the table above.

## License

[GPL-3.0](LICENSE) — Copyright (C) 2026 0rphen
