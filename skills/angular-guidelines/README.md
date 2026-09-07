# angular-guidelines

Claude Code skill for Angular architecture and policy: where state lives,
data-access boundaries, component boundaries, and feature structure. Defers
API syntax, forms, and CLI to `angular-developer`/`angular-new-app`, and
folder structure to `front-guidelines` when installed.

Part of the `ia-utilities` skill package. To activate it in a Claude Code
session, symlink it into the skills directory:

```sh
ln -s "$(pwd)/ia-utilities/skills/angular-guidelines" ~/.claude/skills/angular-guidelines
```

See `SKILL.md` for the rules, and `references/` for the detailed contracts:

- `references/delegation.md` — version detection, ownership map, precedence, standalone fallback tree.
- `references/state.md` — state ladder, store contract, signals vs RxJS, effect rules.
- `references/boundaries.md` — data layer, structure/DI binding, smart/dumb presentation rules.
- `references/antipatterns.md` — anti-patterns and fixes.
