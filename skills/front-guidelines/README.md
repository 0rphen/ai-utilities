# front-guidelines

Claude Code skill enforcing a screaming (feature-first) frontend architecture:
repository + datasource data layer, hard smart/dumb component boundaries, and
centralized/token-driven styling.

Part of the `ia-utilities` skill package. To activate it in a Claude Code
session, symlink it into the skills directory:

```sh
ln -s "$(pwd)/ia-utilities/skills/front-guidelines" ~/.claude/skills/front-guidelines
```

See `SKILL.md` for the rules and `references/` for the detailed contracts and
the canonical `orders` example.
