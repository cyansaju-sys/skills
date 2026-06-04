---
name: migrations
description: >
  Migra código, módulos, aplicaciones completas o secciones específicas de cualquier
  tecnología, framework o lenguaje de programación a otro. Úsala siempre que el usuario
  mencione palabras como "migrar", "portar", "convertir", "reescribir en", "pasar de X a Y",
  "traducir código" o cualquier variante. Aplica tanto a migraciones pequeñas (una función,
  un componente) como grandes (toda una aplicación). Siempre usa archivos del proyecto como
  ejemplos concretos para generar código idiomático en el destino.
---

# Skill: Generador de Migraciones

## Propósito

Migrar código fuente de una tecnología origen a una tecnología destino, preservando el
comportamiento, adaptando los patrones al estilo idiomático del destino, y explicando
cada decisión de traducción relevante.

---

## Proceso

### 1. Entender el Contexto de la Migración

Antes de generar cualquier código, identifica con certeza:

**Sobre el origen:**
- Tecnología / framework / lenguaje (ej: Express.js, Django, Laravel, Rails)
- Versión si es relevante (ej: React 16 vs React 18)
- Patrones usados (ej: clases, hooks, MVC, repositorios)
- Archivos concretos a migrar (rutas, nombres)

**Sobre el destino:**
- Tecnología / framework / lenguaje objetivo
- Versión objetivo si aplica
- Convenciones idiomáticas del destino (ej: FastAPI usa async por defecto, Go usa interfaces implícitas)

**Sobre el alcance:**
- ¿Es una función, un módulo, un servicio, o la aplicación completa?
- ¿Hay dependencias externas que también deben migrarse o reemplazarse?
- ¿Debe mantenerse la misma estructura de archivos o puede reorganizarse?

Si alguno de estos puntos no está claro y no se puede inferir del código, haz **una sola
pregunta** antes de continuar.

---

### 2. Analizar el Código Origen

Antes de traducir, lee y comprende el código fuente:

- Identifica qué hace cada pieza (comportamiento, no solo sintaxis)
- Detecta patrones de diseño usados (singleton, factory, repository, etc.)
- Marca dependencias externas que necesitan equivalente en el destino
- Identifica código que no tiene traducción directa (requiere refactor)

**Regla clave:** migrar es traducir comportamiento, no sintaxis. El código destino debe
hacer lo mismo, pero verse como si hubiera sido escrito originalmente en esa tecnología.

---

### 3. Mapear Equivalencias

Antes de escribir, construye mentalmente (o explícitamente si es complejo) el mapa:

| Origen | Destino | Notas |
|--------|---------|-------|
| `express.Router()` | `APIRouter()` de FastAPI | Misma idea, distinta sintaxis |
| `middleware` con `next()` | `Depends()` en FastAPI | Diferente paradigma |
| `req.body` | parámetro con `Body(...)` | Validación explícita en destino |
| `npm install X` | `pip install Y` | Encontrar equivalente, no solo copiar nombre |

Incluye este mapa en el output si tiene 3 o más equivalencias no triviales.

---

### 4. Generar el Código Migrado

Escribe el código destino siguiendo estas reglas:

**Idiomático primero:** usa los patrones, convenciones y herramientas estándar del
destino. No transliteres: si el origen usa callbacks y el destino usa async/await,
usa async/await.

**Misma lógica de negocio:** el comportamiento debe ser idéntico. Si hay diferencias
inevitables, documéntalas explícitamente.

**Estructura de archivos:** sugiere la estructura de archivos equivalente en el destino
si difiere del origen. Ejemplo:

```
Origen (Express):          Destino (FastAPI):
src/
  routes/users.js    →     routers/users.py
  models/User.js     →     models/user.py
  middleware/auth.js →     dependencies/auth.py
```

**Dependencias:** lista los paquetes/librerías necesarios en el destino con el comando
de instalación exacto.

---

### 5. Explicar las Decisiones de Migración

Después del código, incluye una sección **"Decisiones de migración"** que explique:

- Equivalencias no obvias (¿por qué `X` se convirtió en `Y`?)
- Cambios de paradigma (callbacks → promesas, herencia → composición, etc.)
- Funcionalidad que no existe en el destino y cómo se suplió
- Diferencias de comportamiento inevitables (si las hay)
- Lo que se mejoró aprovechando las fortalezas del destino

**Formato:**

```
### Decisiones de migración

**1. [Concepto origen] → [Concepto destino]**
→ [Explicación de por qué y cómo se hizo la traducción]

**2. [Dependencia X] reemplazada por [Dependencia Y]**
→ [Razón del reemplazo y diferencias de comportamiento si las hay]
```

---

### 6. Verificación y Pasos Siguientes

Al final de cada migración, incluye:

**Checklist de verificación:**
- [ ] Comportamiento idéntico al original en los casos principales
- [ ] Casos borde manejados (errores, valores nulos, timeouts)
- [ ] Dependencias listadas con versiones
- [ ] Variables de entorno / configuración equivalentes documentadas

**Pasos para ejecutar:**
Proporciona los comandos exactos para instalar dependencias y ejecutar el código migrado:

```bash
# Ejemplo para destino Python/FastAPI
pip install fastapi uvicorn
uvicorn main:app --reload
```

**Qué probar primero:** sugiere 2-3 casos de prueba manuales o unitarios para
verificar que la migración es correcta.

---

## Tipos de Migración Comunes

### Framework web completo (ej: Express → FastAPI)
- Migrar rutas, middleware, modelos, configuración y entrypoint
- Reorganizar estructura de archivos si es necesario
- Reemplazar ecosistema de paquetes (npm → pip, etc.)

### Componente UI (ej: Vue → React, React clase → hooks)
- Preservar props, eventos y estado
- Adaptar ciclo de vida al equivalente del destino
- Mantener la misma API pública del componente

### Acceso a datos (ej: Sequelize → SQLAlchemy, Mongoose → Motor)
- Traducir modelos y esquemas
- Adaptar queries al ORM/ODM destino
- Verificar comportamiento de transacciones y relaciones

### Script / utilidad (ej: Bash → Python, JS → Go)
- Preservar inputs/outputs exactos (stdin/stdout, archivos, exit codes)
- Adaptar manejo de errores al idioma del destino

---

## Notas

- Si el usuario proporciona rutas de archivos (`@/ruta`), léelos antes de generar código.
- Si la migración es grande (> 5 archivos), propón un plan por fases antes de empezar.
- Si hay partes que no deben migrarse (ej: base de datos, infraestructura), menciónalo.
- Prefiere claridad sobre brevedad: es mejor un código migrado más verboso pero correcto
  que uno compacto que oculta diferencias de comportamiento.

## Integración con clean-code

Si el usuario pide explícitamente "código limpio", "no copies la lógica", "refactoriza
mientras migras" o similar, **aplica también la skill `clean-code`**:

- No transliteres la lógica original: reescríbela con los principios de Clean Code
- Aprovecha la migración para mejorar nombres, reducir funciones largas, eliminar
  duplicación y aplicar SOLID en el destino
- En la sección "Decisiones de migración", incluye también los principios de Clean Code
  aplicados (igual que haría la skill `clean-code` en su sección "Principios aplicados")
