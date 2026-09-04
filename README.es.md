# ia-utilities

*[Read in English](README.md)*

Una colección de skills y utilidades para [Claude Code](https://claude.ai/code).

## Qué hay aquí

| Skill | Descripción |
| --- | --- |
| [`css-guidelines`](skills/css-guidelines/) | Reglas modernas de autoría CSS/SCSS — cascade layers, ubicación según CUBE CSS, grid/container queries, propiedades lógicas, design tokens y un sistema de color OKLCH por canales L/C/H. |
| [`front-guidelines`](skills/front-guidelines/) | Reglas de arquitectura frontend agnósticas de framework — estructura orientada a features (screaming architecture), capa de datos repository/datasource, separación estricta smart/dumb, escalado por niveles (small/medium/large), y estilos centralizados basados en tokens. |
| [`angular-guidelines`](skills/angular-guidelines/) | Reglas de arquitectura y política específicas de Angular — ubicación del estado, fronteras de acceso a datos, límites de componentes, y política de DI/routing/change-detection. Independiente de versión; complementaria a `front-guidelines`. |

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
  css-guidelines/     # reglas de autoría CSS/SCSS
  front-guidelines/   # arquitectura frontend agnóstica de framework
  angular-guidelines/ # arquitectura y política específicas de Angular
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
