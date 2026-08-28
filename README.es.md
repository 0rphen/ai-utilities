# ia-utilities

*[Read in English](README.md)*

Una colección de skills y utilidades para [Claude Code](https://claude.ai/code).

## Qué hay aquí

| Skill | Descripción |
| --- | --- |
| [`css-guidelines`](skills/css-guidelines/) | Reglas modernas de autoría CSS/SCSS — cascade layers, ubicación según CUBE CSS, grid/container queries, propiedades lógicas, design tokens y un sistema de color OKLCH por canales L/C/H. |
| [`frontend-arch`](skills/frontend-arch/) | Reglas de arquitectura frontend agnósticas de framework — capas y dirección de dependencias, estructura de carpetas orientada a features, descomposición de componentes con atomic design, y ubicación del estado de servidor/cliente. |

## Instalación

### Una skill suelta

Clona este repo y copia (o symlinkea) la skill que quieras dentro de tu directorio de skills:

```sh
git clone <this-repo-url> ia-utilities
ln -s "$(pwd)/ia-utilities/skills/css-guidelines" ~/.claude/skills/css-guidelines
```

### Todas las skills

```sh
git clone <this-repo-url> ia-utilities
for skill in ia-utilities/skills/*/; do
  ln -s "$(pwd)/$skill" ~/.claude/skills/"$(basename "$skill")"
done
```

Una vez instalada, cada skill se auto-carga según su campo `description` del frontmatter — Claude Code decide cuándo es relevante, no hace falta invocarla por nombre.

## Estructura del repositorio

```
skills/
  <skill-name>/
    SKILL.md          # reglas y frontmatter (name, description, allowed-tools) — fuente de verdad
    references/*.md    # docs de detalle que la skill enlaza bajo demanda, no se precargan
    README.md          # notas de instalación/contenido para humanos, de esa skill
```

Cada skill vive en su propio directorio bajo `skills/`. `SKILL.md` es lo que Claude Code lee para decidir cuándo y cómo aplicar la skill; cualquier cosa en `references/` se carga solo cuando `SKILL.md` enlaza hacia ella.

## Añadir una skill

1. Crea un directorio nuevo bajo `skills/<nombre>/`.
2. Añade un `SKILL.md` con frontmatter `name`, `description` y `allowed-tools`, seguido de las reglas.
3. Opcionalmente añade `references/*.md` para detalle que la skill pueda enlazar bajo demanda, y un `README.md` para quien navegue el repo.
4. Añade una fila a la tabla de arriba.

## Licencia

[GPL-3.0](LICENSE) — Copyright (C) 2026 0rphen
