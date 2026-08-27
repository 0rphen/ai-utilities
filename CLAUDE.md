# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository purpose

This repo is a distributable Claude Code **skill package**, not an application — there is no build, lint, or test tooling. It currently contains one skill, `skills/css-guidelines`, meant to be cloned or copied into a user's `~/.claude/skills/` directory (see `skills/css-guidelines/README.md`).

## Structure

- `skills/css-guidelines/SKILL.md` — the skill's frontmatter (`name`, `description`, `allowed-tools`) and the full rule set: cascade layers, CUBE CSS placement, layout/responsiveness, units, tokens, color system, nesting, accessibility, troubleshooting, and hard constraints.
- `skills/css-guidelines/references/*.md` — detail docs the skill links to on demand (not preloaded): `tokens.md`, `layout.md`, `cube.md`, `color.md`, `antipatterns.md`.
- `skills/css-guidelines/README.md` — install instructions and a contents index for humans browsing the repo.

## Working on this skill

- `SKILL.md` is the single source of truth; the `references/*.md` files are supporting detail the skill explicitly points into (e.g. "See `references/layout.md`") — keep new rules in the file whose topic they match rather than growing `SKILL.md` unboundedly.
- The skill enforces strict, opinionated CSS rules (no `!important`, no `px` outside a justified hairline, no hex/`rgb()`/`hsl()`/named colors, logical properties only, OKLCH channel-based color, etc.) — when editing `SKILL.md` or the references, the examples and prose must themselves comply with these rules.
- The `description` field in `SKILL.md`'s frontmatter is what Claude Code uses to decide when to auto-load this skill — keep it specific to the trigger conditions (any CSS/SCSS/`<style>` authoring, editing, reviewing, or styling decision) if it's ever revised.
- Section numbering in `SKILL.md` under **Instructions** is referenced elsewhere (e.g. troubleshooting entries assume the reader has read the numbered rules) — renumber carefully if inserting or removing a rule.
