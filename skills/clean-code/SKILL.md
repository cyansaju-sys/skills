---
name: clean-code
description: >
  Generates clean code from scratch following Clean Code (Robert C. Martin), SOLID, DRY,
  KISS, and YAGNI principles for any programming language. Use it whenever the user asks
  to write new code or implement a function, class, module, or any piece of software —
  even if they don't explicitly mention "clean code" / "código limpio". Also applies when
  the user says "write", "create", "implement", "generate", "make me" (Spanish: "escríbeme",
  "crea", "implementa", "genera", "hazme") something code-related. Always explain in
  detail each clean-code principle applied in the solution.
---

# Skill: Clean Code Generator

## Purpose

Generate new code from scratch applying clean-code principles, with detailed explanations
of every design decision made.

Write all explanation sections in the user's language.

---

## Generation Process

### 1. Understand the Requirement

Before writing code, identify:

- **What does it do?** — the expected behavior
- **Who uses it?** — internal callers, public API, CLI, etc.
- **Which language?** — if not specified, choose the most suitable one for the context and justify it
- **What constraints exist?** — performance, dependencies, project style

If the requirement is ambiguous and you cannot reasonably infer the answer, ask
**a single question** before continuing.

---

### 2. Design Before Writing

Briefly think about:

- Names of functions, classes, and variables
- Responsibilities (what does each piece do?)
- Dependencies and how to inject them
- Relevant edge cases

---

### 3. Write the Code

Apply **all** the principles from the catalog below. The code must be:

- Readable without "explanatory" comments
- Demonstrably correct for the obvious edge cases
- Stylistically consistent with the chosen language (idiomatic)

---

### 4. Explain in Detail

After the code, include an **"Applied Principles"** section with:

- The principle's name
- A one-sentence definition
- Where and how it was applied in the specific code

Use the format from the "Explanation Template" section at the end of this file.

---

## Principles Catalog

### Naming

| Rule | Description |
|---|---|
| Intention-revealing names | The name must answer: what is it? what is it for? how is it used? |
| No cryptic abbreviations | `usr` → `user`, `calc` → `calculate`, `tmp` → `temporaryResult` |
| Pronounceable names | Eases communication during code reviews |
| Searchable names | Avoid unnamed magic constants (`86400` → `SECONDS_PER_DAY`) |
| Distinguish concepts | Don't use `data`, `info`, `manager` without a qualifier |
| Verbs for functions | `getUserById()`, `calculateTotal()`, `isValid()` |
| Nouns for classes | `UserRepository`, `OrderProcessor`, `EmailValidator` |

### Functions

| Rule | Description |
|---|---|
| Do one thing | One function = one responsibility. If it needs "and", it's two functions |
| Small | Ideally < 20 lines; if it grows, extract subfunctions |
| One level of abstraction | Don't mix business logic with implementation details in the same function |
| No hidden side effects | The function does only what its name promises |
| Minimal parameters | 0-2 ideal, 3 acceptable, more → use a configuration object |
| No flag arguments | `processUser(user, true)` → two separate functions |
| Return early (Guard Clauses) | Validate and exit before the main logic |

### Classes and Modules

| Rule | Description |
|---|---|
| Single Responsibility (SRP) | One class = one reason to change |
| Open/Closed (OCP) | Open for extension, closed for modification |
| Liskov Substitution (LSP) | Subclasses must be able to replace their superclasses |
| Interface Segregation (ISP) | Small, specific interfaces, not monolithic ones |
| Dependency Inversion (DIP) | Depend on abstractions, not concrete implementations |
| High cohesion | A class's methods operate on the same data |
| Low coupling | Classes know as little as possible about each other |

### Comments

| Rule | Description |
|---|---|
| Code explains itself | Rename instead of commenting what something does |
| Comments = WHY | Reserve comments for non-obvious decisions, trade-offs, or business context |
| No translation comments | `i++; // increment i` is noise |
| No commented-out code | Delete it; version control keeps it |
| Document public APIs | Do document parameters, return values, and exceptions of APIs |

### Structure and Formatting

| Rule | Description |
|---|---|
| DRY (Don't Repeat Yourself) | Extract any duplicated logic |
| KISS (Keep It Simple) | The simplest solution that works correctly |
| YAGNI (You Aren't Gonna Need It) | Don't implement anticipated functionality without a real requirement |
| Separation of concerns | UI, business logic, and data access in separate layers |
| Law of Demeter | An object only talks to its immediate friends |

---

## Explanation Template

Use this structure after each code fragment:

```
### Applied Principles

**1. [Principle Name]**
> [One-sentence definition]

Applied in: `functionName()` / `ClassX` / line N
→ [Specific explanation of how it was applied and why it improves the code]

**2. [Principle Name]**
...
```

Include between 3 and 8 principles. Prioritize the most visible and important ones in the generated code.
If several principles apply in the same place, group them.

---

## Expected Output Example

**User prompt:** "Write me a function that validates an email"

**Output:**

```python
import re

EMAIL_PATTERN = re.compile(
    r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
)

def is_valid_email(email: str) -> bool:
    """
    Checks whether a string has a valid email format (simplified RFC 5322).
    Does not verify that the domain or mailbox exists.
    """
    if not email or not isinstance(email, str):
        return False
    return bool(EMAIL_PATTERN.match(email.strip()))
```

### Applied Principles

**1. Intention-revealing names**
> The name must answer what it does, what it is for, and how it is used.

Applied in: `is_valid_email(email)`
→ The `is_` prefix signals a boolean return. The `email` parameter is self-descriptive.
  `validate()`, `check()`, or `emailFunc()` were avoided.

**2. Named constant instead of a magic value**
> Magic constants hide the code's intent.

Applied in: `EMAIL_PATTERN`
→ The regular expression is extracted into a descriptively named constant and compiled
  once at module level (efficiency), instead of being recompiled on every call.

**3. Guard Clause (early return)**
> Validate error conditions up front and exit before the main logic.

Applied in: the initial `if not email`
→ Handles edge cases (`None`, empty string, wrong type) explicitly and early,
  making the happy path more readable.

**4. Single responsibility**
> The function does exactly one thing.

Applied in: the whole function
→ It only validates format. It doesn't normalize, query a database, or send emails.
  If normalization were needed, it would be a separate `normalize_email()` function.

**5. "Why" comment instead of "what"**
> Comments explain decisions, they don't translate code.

Applied in: the docstring
→ Clarifies the validation scope (format, not existence) — information that
  can't be inferred from the code and that prevents misuse of the function.

---

## Additional Notes

- If the user requests a specific language, use it. If not specified,
  choose the most appropriate one and briefly mention it before generating.
- Adapt the style to the language's idioms: snake_case in Python, camelCase in JS, etc.
- For longer code (> 50 lines), organize the explanation by code sections.
- If there are relevant trade-offs (performance vs. readability, simplicity vs. flexibility),
  mention them explicitly at the end under "Trade-offs Considered".

## Integration with migrations

If the user asks to migrate or port existing code in addition to applying clean code,
**also apply the `migrations` skill** for the technology translation process.
This skill handles quality principles; `migrations` handles equivalence mapping
and the structure of the migration output.
