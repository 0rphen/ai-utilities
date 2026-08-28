# frontend-arch

A Claude Code skill with framework-agnostic frontend architecture rules: layering and dependency direction (domain/application/infrastructure/presentation), feature-first folder structure, atomic-design component decomposition, server/client state placement, and clean-code principles applied inside each layer.

## Install

Clone (or copy) this repo into your skills directory:

```sh
git clone <this-repo-url> ~/.claude/skills/frontend-arch
```

Or add it as a plugin per your Claude Code setup. Once installed, Claude reads `SKILL.md` before scaffolding a frontend project, structuring or refactoring a feature, placing a file or piece of state, splitting an oversized component, or reviewing a codebase's architecture.

## Contents

- `SKILL.md` — the rules: dependency direction, layer responsibilities, feature-first structure, atomic-design decomposition, state placement, boundary enforcement.
- `references/layers.md` — the dependency rule, what belongs in domain/application/infrastructure, a worked end-to-end trace, and the layer collapse table.
- `references/structure.md` — feature-first folder tree, file-suffix naming, barrel-file policy, import-boundary lint config sketch.
- `references/presentation.md` — atomic-design levels crossed with the smart/dumb axis, the templates-have-no-logic rule.
- `references/state.md` — server state vs. client state vs. URL state, loading/error/empty boundary placement.
- `references/clean-code.md` — naming, function size, SRP inside a layer, DRY/KISS, error handling, testability, SOLID.
- `references/antipatterns.md` — antipattern → replacement table, used in audit mode.
