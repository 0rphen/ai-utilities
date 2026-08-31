# front-guidelines

Claude Code skill enforcing a screaming (feature-first) frontend architecture:
repository + datasource data layer, hard smart/dumb component boundaries,
tiered scaling (small/medium/large), and centralized/token-driven styling.

Part of the `ia-utilities` skill package. To activate it in a Claude Code
session, symlink it into the skills directory:

```sh
ln -s "$(pwd)/ia-utilities/skills/front-guidelines" ~/.claude/skills/front-guidelines
```

See `SKILL.md` for the rules, and `references/` for the detailed contracts:

- `references/tiers.md` — project tier declaration, definitions, decision matrix.
- `references/structure.md` — directory tree, naming fallback, barrel rule.
- `references/data-layer.md` — port/impl contract, tiered `data/` shape, cache/TTL/offline.
- `references/presentation.md` — smart/dumb detail, facade necessity test.
- `references/state.md` — state ladder by tier, store encapsulation contract.
- `references/example-orders.md` — canonical end-to-end feature template.
- `references/antipatterns.md` — anti-patterns and fixes.
