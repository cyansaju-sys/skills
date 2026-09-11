---
name: migrations
description: >
  Migrates code, modules, entire applications, or specific sections from any technology,
  framework, or programming language to another. Use it whenever the user mentions words
  like "migrate", "port", "convert", "rewrite in", "move from X to Y", "translate code"
  (Spanish: "migrar", "portar", "convertir", "reescribir en", "pasar de X a Y",
  "traducir código") or any variant. Applies to both small migrations (a function, a
  component) and large ones (an entire application). Always use project files as concrete
  examples to generate idiomatic code in the target.
---

# Skill: Migration Generator

## Purpose

Migrate source code from a source technology to a target technology, preserving
behavior, adapting patterns to the target's idiomatic style, and explaining
every relevant translation decision.

Write all explanation sections in the user's language.

---

## Process

### 1. Understand the Migration Context

Before generating any code, identify with certainty:

**About the source:**
- Technology / framework / language (e.g., Express.js, Django, Laravel, Rails)
- Version if relevant (e.g., React 16 vs. React 18)
- Patterns used (e.g., classes, hooks, MVC, repositories)
- Concrete files to migrate (paths, names)

**About the target:**
- Target technology / framework / language
- Target version if applicable
- Idiomatic conventions of the target (e.g., FastAPI is async by default, Go uses implicit interfaces)

**About the scope:**
- Is it a function, a module, a service, or the entire application?
- Are there external dependencies that also need to be migrated or replaced?
- Must the same file structure be kept, or can it be reorganized?

If any of these points is unclear and cannot be inferred from the code, ask **a single
question** before continuing.

---

### 2. Analyze the Source Code

Before translating, read and understand the source code:

- Identify what each piece does (behavior, not just syntax)
- Detect design patterns used (singleton, factory, repository, etc.)
- Flag external dependencies that need an equivalent in the target
- Identify code with no direct translation (requires refactoring)

**Key rule:** migrating means translating behavior, not syntax. The target code must
do the same thing, but look as if it had originally been written in that technology.

---

### 3. Map Equivalences

Before writing, build the map mentally (or explicitly if complex):

| Source | Target | Notes |
|--------|--------|-------|
| `express.Router()` | FastAPI `APIRouter()` | Same idea, different syntax |
| `middleware` with `next()` | `Depends()` in FastAPI | Different paradigm |
| `req.body` | parameter with `Body(...)` | Explicit validation in target |
| `npm install X` | `pip install Y` | Find the equivalent, don't just copy the name |

Include this map in the output if it has 3 or more non-trivial equivalences.

---

### 4. Generate the Migrated Code

Write the target code following these rules:

**Idiomatic first:** use the target's standard patterns, conventions, and tools.
Don't transliterate: if the source uses callbacks and the target uses async/await,
use async/await.

**Same business logic:** behavior must be identical. If there are unavoidable
differences, document them explicitly.

**File structure:** suggest the equivalent file structure in the target
if it differs from the source. Example:

```
Source (Express):          Target (FastAPI):
src/
  routes/users.js    →     routers/users.py
  models/User.js     →     models/user.py
  middleware/auth.js →     dependencies/auth.py
```

**Dependencies:** list the packages/libraries needed in the target with the exact
install command.

---

### 5. Explain the Migration Decisions

After the code, include a **"Migration Decisions"** section explaining:

- Non-obvious equivalences (why did `X` become `Y`?)
- Paradigm shifts (callbacks → promises, inheritance → composition, etc.)
- Functionality missing in the target and how it was replaced
- Unavoidable behavior differences (if any)
- What was improved by leveraging the target's strengths

**Format:**

```
### Migration Decisions

**1. [Source concept] → [Target concept]**
→ [Explanation of why and how the translation was done]

**2. [Dependency X] replaced by [Dependency Y]**
→ [Reason for the replacement and behavior differences, if any]
```

---

### 6. Verification and Next Steps

At the end of every migration, include:

**Verification checklist:**
- [ ] Behavior identical to the original in the main cases
- [ ] Edge cases handled (errors, null values, timeouts)
- [ ] Dependencies listed with versions
- [ ] Equivalent environment variables / configuration documented

**Steps to run:**
Provide the exact commands to install dependencies and run the migrated code:

```bash
# Example for a Python/FastAPI target
pip install fastapi uvicorn
uvicorn main:app --reload
```

**What to test first:** suggest 2-3 manual or unit test cases to
verify the migration is correct.

---

## Common Migration Types

### Full web framework (e.g., Express → FastAPI)
- Migrate routes, middleware, models, configuration, and entrypoint
- Reorganize the file structure if needed
- Replace the package ecosystem (npm → pip, etc.)

### UI component (e.g., Vue → React, React class → hooks)
- Preserve props, events, and state
- Adapt the lifecycle to the target's equivalent
- Keep the same public API for the component

### Data access (e.g., Sequelize → SQLAlchemy, Mongoose → Motor)
- Translate models and schemas
- Adapt queries to the target ORM/ODM
- Verify transaction and relationship behavior

### Script / utility (e.g., Bash → Python, JS → Go)
- Preserve exact inputs/outputs (stdin/stdout, files, exit codes)
- Adapt error handling to the target's idioms

---

## Notes

- If the user provides file paths (`@path`), read them before generating code.
- If the migration is large (> 5 files), propose a phased plan before starting.
- If there are parts that should not be migrated (e.g., database, infrastructure), say so.
- Prefer clarity over brevity: verbose but correct migrated code is better
  than compact code that hides behavior differences.

## Integration with clean-code

If the user explicitly asks for "clean code", "don't copy the logic", "refactor while
migrating" (Spanish: "código limpio", "no copies la lógica", "refactoriza mientras
migras") or similar, **also apply the `clean-code` skill**:

- Don't transliterate the original logic: rewrite it using Clean Code principles
- Use the migration to improve names, shorten long functions, remove
  duplication, and apply SOLID in the target
- In the "Migration Decisions" section, also include the Clean Code principles
  applied (just as the `clean-code` skill does in its "Applied Principles" section)
