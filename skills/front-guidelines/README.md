# front-guidelines

Claude Code skill enforcing a screaming (feature-first) frontend architecture:
a `domain`/`structure` data layer (declarations vs. implementations), a
mandatory facade, hard smart/dumb component boundaries, and centralized/
token-driven styling.

Part of the `ia-utilities` skill package. To activate it in a Claude Code
session, symlink it into the skills directory:

```sh
ln -s "$(pwd)/ia-utilities/skills/front-guidelines" ~/.claude/skills/front-guidelines
```

See `SKILL.md` for the rules, and `references/` for the detailed contracts:

- `references/structure.md` — directory tree, naming fallback, barrel rule, `core`/`shared`/`utils` criteria.
- `references/data-layer.md` — `domain`/`structure` contract, mapper, cache/TTL placement.
- `references/presentation.md` — smart/dumb detail, mandatory facade contract.
- `references/state.md` — state ladder, store encapsulation contract.
- `references/example-orders.md` — canonical end-to-end feature template.
- `references/antipatterns.md` — anti-patterns and fixes.
