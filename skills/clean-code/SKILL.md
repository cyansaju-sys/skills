---
name: clean-code
description: >
  Genera código limpio desde cero siguiendo principios de Clean Code (Robert C. Martin),
  SOLID, DRY, KISS y YAGNI para cualquier lenguaje de programación. Úsala siempre que
  el usuario pida escribir código nuevo, implementar una función, clase, módulo, o
  cualquier pieza de software — incluso si no menciona explícitamente "código limpio".
  Esta skill también aplica cuando el usuario dice "escríbeme", "crea", "implementa",
  "genera" o "hazme" algo relacionado con código. Siempre explica en detalle cada
  principio de código limpio aplicado en la solución.
---

# Skill: Generador de Código Limpio

## Propósito

Generar código nuevo desde cero aplicando principios de código limpio, con explicaciones
detalladas de cada decisión de diseño tomada.

---

## Proceso de Generación

### 1. Entender el Requerimiento

Antes de escribir código, identifica:

- **¿Qué hace?** — el comportamiento esperado
- **¿Quién lo usa?** — llamadores internos, API pública, CLI, etc.
- **¿Qué lenguaje?** — si no se especifica, elige el más adecuado al contexto y justifícalo
- **¿Qué restricciones hay?** — rendimiento, dependencias, estilo del proyecto

Si el requerimiento es ambiguo y no puedes inferir la respuesta de forma razonable, haz
**una sola pregunta** antes de continuar.

---

### 2. Diseñar Antes de Escribir

Piensa brevemente en:

- Nombres de funciones, clases y variables
- Responsabilidades (¿qué hace cada pieza?)
- Dependencias y cómo inyectarlas
- Casos borde relevantes

---

### 3. Escribir el Código

Aplica **todos** los principios del catálogo de abajo. El código debe ser:

- Legible sin comentarios de "explicación"
- Probablemente correcto en los casos borde obvios
- Coherente en estilo con el lenguaje elegido (idiomático)

---

### 4. Explicar en Detalle

Después del código, incluye una sección **"Principios aplicados"** con:

- El nombre del principio
- Una frase de qué es
- Dónde y cómo se aplicó en el código específico

Usa el formato de la sección de "Plantilla de Explicación" al final de este archivo.

---

## Catálogo de Principios

### Nomenclatura

| Regla | Descripción |
|---|---|
| Nombres que revelan intención | El nombre debe responder: ¿qué es? ¿para qué sirve? ¿cómo se usa? |
| Sin abreviaciones crípticas | `usr` → `user`, `calc` → `calculate`, `tmp` → `temporaryResult` |
| Nombres pronunciables | Facilita la comunicación en revisiones de código |
| Nombres buscables | Evita constantes mágicas sin nombre (`86400` → `SECONDS_PER_DAY`) |
| Distinguir conceptos | No usar `data`, `info`, `manager` sin calificador |
| Verbos para funciones | `getUserById()`, `calculateTotal()`, `isValid()` |
| Sustantivos para clases | `UserRepository`, `OrderProcessor`, `EmailValidator` |

### Funciones

| Regla | Descripción |
|---|---|
| Hacer una sola cosa | Una función = una responsabilidad. Si necesita "y", es dos funciones |
| Pequeñas | Idealmente < 20 líneas; si crece, extraer subfunciones |
| Un nivel de abstracción | No mezclar lógica de negocio con detalles de implementación en la misma función |
| Sin efectos secundarios ocultos | La función solo hace lo que su nombre promete |
| Parámetros mínimos | 0-2 ideal, 3 aceptable, más → usar objeto de configuración |
| No usar flags como parámetro | `processUser(user, true)` → dos funciones separadas |
| Retornar temprano (Guard Clauses) | Validar y salir antes de la lógica principal |

### Clases y Módulos

| Regla | Descripción |
|---|---|
| Single Responsibility (SRP) | Una clase = una razón para cambiar |
| Open/Closed (OCP) | Abierta para extensión, cerrada para modificación |
| Liskov Substitution (LSP) | Las subclases deben poder reemplazar a las superclases |
| Interface Segregation (ISP) | Interfaces pequeñas y específicas, no monolíticas |
| Dependency Inversion (DIP) | Depender de abstracciones, no de implementaciones concretas |
| Cohesión alta | Los métodos de una clase trabajan sobre los mismos datos |
| Acoplamiento bajo | Las clases conocen lo mínimo posible de otras |

### Comentarios

| Regla | Descripción |
|---|---|
| El código se explica solo | Renombra en lugar de comentar qué hace algo |
| Comentarios = POR QUÉ | Reservar comentarios para decisiones no obvias, trade-offs, o contexto de negocio |
| Sin comentarios de traducción | `i++; // incrementa i` es ruido |
| Sin código comentado | Eliminarlo; el control de versiones lo guarda |
| Documentar APIs públicas | Sí documentar parámetros, retornos y excepciones de APIs |

### Estructura y Formato

| Regla | Descripción |
|---|---|
| DRY (Don't Repeat Yourself) | Extraer cualquier lógica duplicada |
| KISS (Keep It Simple) | La solución más simple que funcione correctamente |
| YAGNI (You Aren't Gonna Need It) | No implementar funcionalidad anticipada sin requerimiento real |
| Separación de concerns | UI, lógica de negocio, acceso a datos en capas distintas |
| Ley de Demeter | Un objeto solo habla con sus amigos directos |

---

## Plantilla de Explicación

Usa esta estructura después de cada fragmento de código:

```
### Principios aplicados

**1. [Nombre del Principio]**
> [Definición en una oración]

Aplicado en: `nombreFuncion()` / `ClaseX` / línea N
→ [Explicación específica de cómo se aplicó y por qué mejora el código]

**2. [Nombre del Principio]**
...
```

Incluye entre 3 y 8 principios. Prioriza los más visibles e importantes en el código generado.
Si se aplican varios principios en el mismo lugar, agrúpalos.

---

## Ejemplo de Output Esperado

**Prompt del usuario:** "Escríbeme una función que valide un email"

**Output:**

```python
import re

EMAIL_PATTERN = re.compile(
    r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
)

def is_valid_email(email: str) -> bool:
    """
    Verifica si una cadena tiene formato de email válido (RFC 5322 simplificado).
    No verifica existencia del dominio ni del buzón.
    """
    if not email or not isinstance(email, str):
        return False
    return bool(EMAIL_PATTERN.match(email.strip()))
```

### Principios aplicados

**1. Nombres que revelan intención**
> El nombre debe responder qué hace, para qué sirve y cómo se usa.

Aplicado en: `is_valid_email(email)`
→ El prefijo `is_` indica que retorna un booleano. El parámetro `email` es auto-descriptivo.
  No se usó `validate()`, `check()` o `emailFunc()`.

**2. Constante nombrada en lugar de valor mágico**
> Las constantes mágicas ocultan la intención del código.

Aplicado en: `EMAIL_PATTERN`
→ La expresión regular se extrae a una constante con nombre descriptivo y se compila
  una sola vez a nivel de módulo (eficiencia), en lugar de recompilarla en cada llamada.

**3. Guard Clause (retorno temprano)**
> Validar condiciones de error al inicio y salir, antes de la lógica principal.

Aplicado en: el `if not email` inicial
→ Maneja los casos borde (`None`, cadena vacía, tipo incorrecto) de forma explícita
  y temprana, haciendo el flujo feliz más legible.

**4. Una sola responsabilidad**
> La función hace exactamente una cosa.

Aplicado en: toda la función
→ Solo valida formato. No normaliza, no busca en base de datos, no envía correos.
  Si se necesitara normalizar, sería una función separada `normalize_email()`.

**5. Comentario de "por qué" en lugar de "qué"**
> Los comentarios explican decisiones, no traducen código.

Aplicado en: el docstring
→ Aclara el alcance de la validación (formato, no existencia) — información que
  no se puede inferir del código, y que evita mal uso de la función.

---

## Notas Adicionales

- Si el usuario pide código en un lenguaje específico, úsalo. Si no especifica,
  elige el más apropiado y menciónalo brevemente antes de generar.
- Adapta el estilo al idioma del lenguaje: snake_case en Python, camelCase en JS, etc.
- Para código más largo (> 50 líneas), organiza la explicación por secciones del código.
- Si hay trade-offs relevantes (rendimiento vs legibilidad, simplicidad vs flexibilidad),
  menciónalos explícitamente al final bajo "Trade-offs considerados".

## Integración con migrations

Si el usuario pide migrar o portar código existente además de aplicar código limpio,
**aplica también la skill `migrations`** para el proceso de traducción tecnológica.
Esta skill se encarga de los principios de calidad; `migrations` se encarga del mapeo
de equivalencias y la estructura del output de migración.
