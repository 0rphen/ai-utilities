# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository purpose

This repo is a distributable Claude Code **skill package**, not an application — there is no build, lint, or test tooling. It contains two skills, meant to be cloned or copied into a user's `~/.claude/skills/` directory (see each skill's own `README.md`):

- `skills/css-guidelines` — CSS/SCSS authoring rules.
- `skills/front-guidelines` — framework-agnostic frontend architecture rules (feature-first/screaming structure, repository/datasource data layer, smart/dumb component split, centralized styling).

## Structure

Every skill follows the same shape:

```
skills/<skill-name>/
  SKILL.md          # frontmatter (name, description, allowed-tools) + the full numbered rule set — source of truth
  references/*.md    # detail docs the skill links to on demand, not preloaded into context by default
  README.md          # install instructions and a contents index for humans browsing the repo
```

- `skills/css-guidelines/references/*.md`: `tokens.md`, `layout.md`, `cube.md`, `color.md`, `antipatterns.md` — cascade layers, CUBE CSS placement, layout/responsiveness, tokens, OKLCH color, and the antipattern table.
- `skills/front-guidelines/references/*.md`: `structure.md`, `data-layer.md`, `presentation.md`, `example-orders.md`, `antipatterns.md` — directory tree and naming fallback table, repository/datasource port contracts, the smart/dumb + facade rules, a canonical end-to-end feature example, and the antipattern table.

## Working on a skill

- `SKILL.md` is the single source of truth for its skill; the `references/*.md` files are supporting detail the skill explicitly points into (e.g. "See `references/layout.md`") — keep new rules in the file whose topic they match rather than growing `SKILL.md` unboundedly. Both skills target well under 500 lines for `SKILL.md`, in practice under ~120.
- `skills/css-guidelines` follows a house style: `SKILL.md` opens with a human-title H1 and a "not X, not Y — Z" positioning paragraph, then `## Overview` → `## Instructions` (flat numbered list, `**bold imperative label**: body`, no nesting) → `## Best Practices` → `## Troubleshooting` (`###` symptom-as-experienced headings + `**Fix**:`) → `## Constraints and Warnings` (bolded prohibitions) → `## References` (one bullet per reference file, with an "open before/when…" clause). Its reference files use `# <Skill Name> — <Topic>` H1s, H2-only sections separated by `---`, and `<!-- ✅ -->` / `<!-- ❌ never — reason -->` HTML-comment markers around do/don't code pairs. `skills/front-guidelines` intentionally uses a leaner shape optimized for a small always-loaded `SKILL.md` (`## Overview` → `## Instructions` → `## Constraints` → `## References`; anti-patterns live in their own `references/antipatterns.md` table instead of a `SKILL.md` section) — when editing either skill, match that skill's own existing shape rather than importing the other's.
- `skills/css-guidelines` enforces strict, opinionated CSS rules (no `!important`, no `px` outside a justified hairline, no hex/`rgb()`/`hsl()`/named colors, logical properties only, OKLCH channel-based color, etc.) — when editing its `SKILL.md` or references, the examples and prose must themselves comply with these rules.
- `skills/front-guidelines` is framework-agnostic by design — when editing its `SKILL.md` or `references/structure.md`, `data-layer.md`, `presentation.md`, keep examples free of framework-specific imports/APIs outside an explicitly labeled aside. `references/example-orders.md` is the one exception on purpose: it carries a labeled React example (`useState`, `useQuery`) to make the canonical feature template concrete, so a blanket grep for those APIs across the whole skill will false-positive on that file — exclude it explicitly: `grep -riE '@Component|@Injectable|useState|ngOnInit|HttpClient' skills/front-guidelines/ | grep -v example-orders.md` should return nothing.
- The `description` field in each `SKILL.md`'s frontmatter is what Claude Code uses to decide when to auto-load that skill — keep it specific to the trigger conditions (concrete verbs, artifacts, and topics) if it's ever revised.
- Section numbering in `SKILL.md` under **Instructions** is referenced elsewhere (e.g. troubleshooting entries or cross-references assume the reader has read the numbered rules in order) — renumber carefully if inserting or removing a rule, and update every reference to a shifted number.
- Root `README.md` / `README.es.md` each carry one table row per skill — add/update both (English and Spanish) when a skill is added or its description changes.
