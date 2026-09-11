# Skills para Claude Code

Colección personal de [skills](https://docs.claude.com/en/docs/claude-code/skills) para Claude Code.
Cada skill es un conjunto de instrucciones que Claude carga automáticamente cuando la
tarea encaja con su descripción, o manualmente con `/<nombre-de-la-skill>`.

## Skills disponibles

| Skill | Qué hace | Se activa cuando pides… |
|---|---|---|
| [`clean-code`](skills/clean-code/SKILL.md) | Genera código nuevo aplicando Clean Code, SOLID, DRY, KISS y YAGNI, y explica cada principio aplicado. | "escríbeme", "crea", "implementa", "genera", "hazme" código |
| [`migrations`](skills/migrations/SKILL.md) | Migra código entre lenguajes, frameworks o tecnologías (ej. Express → FastAPI, Vue → React), preservando el comportamiento y usando patrones idiomáticos del destino. | "migrar", "portar", "convertir", "reescribir en", "pasar de X a Y" |
| [`migrations-android`](skills/migrations-android/SKILL.md) | Actualiza apps Android a `targetSdk`/`compileSdk` 36 (Android 16): edge-to-edge, insets, predictive back y cambios de comportamiento obligatorios. Cubre Views (XML) y Jetpack Compose. | "API 36", "Android 16", "edge-to-edge", "insets", "predictive back" |
Universales
### Skills combinadas

`clean-code` y `migrations` están pensadas para usarse juntas: si pides migrar código
**y** además "código limpio" o "refactoriza mientras migras", `migrations` se encarga del
mapeo de equivalencias y `clean-code` de la calidad del código resultante.

## Reglas de desarrollo

Además de las skills, [`claude.md`](claude.md) define las reglas que Claude debe seguir
en **cualquier** tarea. Resumen (el archivo es la fuente de verdad):

| # | Regla | En resumen |
|---|---|---|
| 1 | Restricción de archivos | Solo modifica el archivo seleccionado; para tocar otros, pide permiso. |
| 2 | Claridad multi-lenguaje | Código limpio y legible, siguiendo la guía de estilo oficial del lenguaje (PEP 8, Standard/AirBnB, Oracle…). |
| 3 | Seguridad adaptativa | Sin vulnerabilidades: evitar inyecciones, XSS/CSRF, sanitizar inputs, nada de credenciales en el código. |
| 4 | Concisión | Cambios pequeños → mostrar solo lo modificado, no reescribir archivos completos. |
| 5 | Seguridad del entorno | Confirmar antes de comandos que alteren el sistema, variables globales o instalen paquetes. |
| 6 | Explicación previa | Explicar brevemente qué se va a cambiar antes de entregar el código. |
| 7 | Eficiencia de tokens | Respuestas concisas, leer solo lo necesario y no repetir contenido, sin sacrificar validaciones ni seguridad. |

Las reglas complementan a las skills: `clean-code` refuerza la regla 2 y las skills de
migración deben respetar las reglas 1, 4 y 5 al tocar archivos o instalar dependencias.

## Instalación

Claude Code busca skills en dos ubicaciones:

- **Personal** (todos tus proyectos): `~/.claude/skills/<nombre>/SKILL.md`
- **Proyecto** (solo ese repo): `.claude/skills/<nombre>/SKILL.md`

### Opción 1: enlaces simbólicos (recomendado)

Así los cambios que hagas en este repo se reflejan al instante:

```bash
mkdir -p ~/.claude/skills
for skill in skills/*/; do
  ln -sfn "$(realpath "$skill")" ~/.claude/skills/"$(basename "$skill")"
done
```

### Opción 2: copiar

```bash
mkdir -p ~/.claude/skills
cp -r skills/* ~/.claude/skills/
```

### Reglas de desarrollo

Claude Code carga automáticamente `~/.claude/CLAUDE.md` en todas las sesiones. Para
aplicar las reglas globalmente:

```bash
ln -sfn "$(realpath claude.md)" ~/.claude/CLAUDE.md
```

> Si ya tienes un `~/.claude/CLAUDE.md`, no lo sobrescribas: copia las reglas dentro de él.

Reinicia la sesión de Claude Code para que detecte las skills y reglas nuevas.

## Uso

- **Automático:** describe la tarea normalmente (ej. *"migra este controlador de Express a
  FastAPI"*) y Claude elige la skill según su `description`.
- **Manual:** invócala directamente, ej. `/migrations-android`.

## Estructura del repositorio

```
.
├── README.md
├── claude.md               # reglas de desarrollo para Claude
├── setting.json            # configuración de Claude Code (vacía por ahora)
└── skills/
    ├── clean-code/
    │   └── SKILL.md
    ├── migrations/
    │   └── SKILL.md
    └── migrations-android/
        └── SKILL.md
```

## Crear una skill nueva

1. Crea la carpeta `skills/<nombre>/` con un archivo `SKILL.md`.
2. Añade el frontmatter con `name` (igual al nombre de la carpeta) y `description`:

   ```markdown
   ---
   name: mi-skill
   description: >
     Qué hace la skill y cuándo usarla. Incluye las palabras clave con las que
     el usuario suele pedir esta tarea: de esto depende que se active sola.
   ---

   # Skill: Título

   Instrucciones para Claude…
   ```

3. Agrégala a la tabla de [Skills disponibles](#skills-disponibles).
4. Si usas enlaces simbólicos, vuelve a ejecutar el comando de instalación.
