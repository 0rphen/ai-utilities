# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository purpose

This repo is a distributable Claude Code **skill package**, not an application — there is no build, lint, or test tooling. It contains two skills, meant to be cloned or copied into a user's `~/.claude/skills/` directory (see each skill's own `README.md`):

- `skills/css-guidelines` — CSS/SCSS authoring rules.
- `skills/frontend-arch` — framework-agnostic frontend architecture rules (layering, folder structure, atomic-design decomposition, state placement, clean code).

## Structure

Every skill follows the same shape:

```
skills/<skill-name>/
  SKILL.md          # frontmatter (name, description, allowed-tools) + the full numbered rule set — source of truth
  references/*.md    # detail docs the skill links to on demand, not preloaded into context by default
  README.md          # install instructions and a contents index for humans browsing the repo
```

- `skills/css-guidelines/references/*.md`: `tokens.md`, `layout.md`, `cube.md`, `color.md`, `antipatterns.md` — cascade layers, CUBE CSS placement, layout/responsiveness, tokens, OKLCH color, and the antipattern table.
- `skills/frontend-arch/references/*.md`: `layers.md`, `structure.md`, `presentation.md`, `state.md`, `clean-code.md`, `antipatterns.md` — the domain/application/infrastructure/presentation dependency rule, feature-first folder structure, atomic-design + smart/dumb decomposition, server/client state placement, clean-code principles applied inside a layer, and the antipattern table.

## Working on a skill

- `SKILL.md` is the single source of truth for its skill; the `references/*.md` files are supporting detail the skill explicitly points into (e.g. "See `references/layout.md`") — keep new rules in the file whose topic they match rather than growing `SKILL.md` unboundedly. Both skills target well under 500 lines for `SKILL.md`, in practice under ~120.
- Both skills share a house style: `SKILL.md` opens with a human-title H1 and a "not X, not Y — Z" positioning paragraph, then `## Overview` → `## Instructions` (flat numbered list, `**bold imperative label**: body`, no nesting) → `## Best Practices` → `## Troubleshooting` (`###` symptom-as-experienced headings + `**Fix**:`) → `## Constraints and Warnings` (bolded prohibitions) → `## References` (one bullet per reference file, with an "open before/when…" clause). Reference files use `# <Skill Name> — <Topic>` H1s, H2-only sections separated by `---`, and `<!-- ✅ -->` / `<!-- ❌ never — reason -->` HTML-comment markers around do/don't code pairs (bad examples get a `-bad`/`Bad` suffixed identifier so they can't be copy-pasted as real code). When adding a new skill or reference file, match this shape rather than inventing a new one.
- `skills/css-guidelines` enforces strict, opinionated CSS rules (no `!important`, no `px` outside a justified hairline, no hex/`rgb()`/`hsl()`/named colors, logical properties only, OKLCH channel-based color, etc.) — when editing its `SKILL.md` or references, the examples and prose must themselves comply with these rules.
- `skills/frontend-arch` is framework-agnostic by design — when editing its `SKILL.md` or references, examples must stay free of framework-specific imports/APIs (no `@Component`, `@Injectable`, `useState`, `HttpClient`, etc.) outside an explicitly labeled aside; `grep -riE '@Component|@Injectable|useState|ngOnInit|HttpClient' skills/frontend-arch/` should return nothing.
- The `description` field in each `SKILL.md`'s frontmatter is what Claude Code uses to decide when to auto-load that skill — keep it specific to the trigger conditions (concrete verbs, artifacts, and topics) if it's ever revised.
- Section numbering in `SKILL.md` under **Instructions** is referenced elsewhere (e.g. troubleshooting entries or cross-references assume the reader has read the numbered rules in order) — renumber carefully if inserting or removing a rule, and update every reference to a shifted number.
- Root `README.md` / `README.es.md` each carry one table row per skill — add/update both (English and Spanish) when a skill is added or its description changes.
