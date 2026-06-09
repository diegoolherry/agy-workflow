---
name: code-review
description: "Revisión de código pre-merge. Analizar el diff contra la rama base buscando bugs reales, problemas de seguridad, calidad y completitud. Usar cuando el usuario dice 'revisá este código', 'code review', 'revisá el diff', 'revisá antes de mergear', o cuando está por mergear cambios. Invocar proactivamente cuando el usuario termina una feature o bugfix y menciona que quiere mergearlo."
---

# Code Review

Revisión estructurada del diff antes de mergear. El objetivo es encontrar bugs que pasan el compilador pero rompen en producción.

Leer `docs/architecture.md` si existe antes de empezar — el contexto de dominio cambia lo que es un bug y lo que es comportamiento correcto. Si existe `.agents/docs/architecture/decisions/`, leer los ADRs relevantes: un finding que viola a propósito un patrón documentado en un ADR es más grave que uno que simplemente es descuidado.

---

## Paso 1 — Obtener contexto

Leer el archivo `.agents/docs/progress/impl/impl_{feature}.md` para identificar la lista de archivos que fueron creados o modificados por el implementador en esta feature.

En lugar de usar ramas remotas o comandos `git diff` complejos, tu objetivo es revisar el código fuente de esos archivos específicos en el entorno local. Esto garantiza que la revisión se concentre estrictamente en lo que se trabajó.

Leer el código de esos archivos completamente antes de emitir cualquier finding. No flaggear issues que estén fuera del scope de la feature.

---

## Paso 2 — Detección de scope

Antes de revisar, basándote en los archivos identificados, determina qué tipo de código se está tocando:

Señales a detectar:
- **SCOPE_AUTH** — archivos de autenticación, autorización, tokens, sesiones, passwords
- **SCOPE_DB** — migraciones, queries, acceso a datos, repositorios
- **SCOPE_API** — controllers, endpoints, DTOs, serialización
- **SCOPE_FRONTEND** — vistas, componentes, CSS
- **SCOPE_CONCURRENCY** — async/await, Tasks, threads, locks

---

## Paso 3 — Revisión crítica

Aplicar estas categorías contra el diff. Para cada finding, **citar la línea de código que lo motiva** — si no se puede citar, no es un finding verificado.

### Categorías CRÍTICAS (P1 — bloquean el merge)

**Seguridad y autenticación** (si SCOPE_AUTH)
- Validación de input ausente o bypasseable
- Datos sensibles loggeados o expuestos en respuestas
- Autorización que puede saltarse (falta chequeo de permisos, IDOR)
- Tokens o secrets hardcodeados

**Seguridad de datos** (si SCOPE_DB)
- SQL construido con concatenación de strings (SQL injection)
- Queries que exponen datos de otros usuarios
- Migraciones destructivas sin respaldo contemplado
- Transacciones que pueden dejar el sistema en estado inconsistente

**Concurrencia** (si SCOPE_CONCURRENCY)
- Race conditions en acceso a estado compartido
- async/await mal usado que puede causar deadlock o comportamiento inesperado

**Null/reference safety**
- NullReferenceException posible en path no cubierto
- Valores opcionales usados sin guard

### Categorías INFORMACIONALES (P2 — reportar, no bloquean)

**Completitud de enums y switchs**
- Nuevo valor de enum sin manejar en switchs existentes
- ⚠️ Esta categoría requiere leer código FUERA del diff — buscar con grep todos los archivos que referencian el enum modificado

**Contratos de API**
- Cambios breaking en endpoints existentes (campos removidos, tipos cambiados)
- Respuestas de error no documentadas

**Calidad**
- Lógica duplicada que debería extraerse
- Nombres que no reflejan el comportamiento real
- Métodos que hacen demasiado (violan SRP)
- Manejo de errores ausente o demasiado genérico (`catch (Exception e)`)

**Testing**
- Caso feliz cubierto pero edge cases sin test
- Bug fixeado sin regression test

---

## Paso 4 — Formato de findings

Cada finding sigue este formato:

```
[P1] (confianza: N/10) archivo:línea — descripción del problema
  Código: <línea exacta que lo motiva>
  Fix: <qué cambiar, concreto>
```

```
[P2] (confianza: N/10) archivo:línea — descripción
  Código: <línea exacta>
  Fix: <recomendación>
```

**Escala de confianza:**
- 9-10: Leí el código específico. Bug concreto demostrado.
- 7-8: Match de patrón de alta confianza. Muy probablemente correcto.
- 5-6: Puede ser falso positivo. Incluir con advertencia: "verificar si es realmente un problema".
- 3-4: No incluir en el reporte principal. Ir al apéndice si algo.
- 1-2: No reportar salvo que la severidad sea P1.

**Regla:** si no podés citar la línea que motiva el finding, forzar confianza a 4 o menos.

---

## Paso 5 — Reporte final

Agrega (append) al final del archivo `.agents/docs/progress/review/review_{feature}.md` generado previamente por `spec-review`, el resultado de tu análisis de calidad:

```markdown

## 3. Code Review (Calidad y Seguridad)
**Scope detectado:** [auth] [db] [api] [frontend] [concurrencia]

### Findings Críticos (P1)
*[Lista o "Ninguno"]*

### Findings Informativos (P2)
*[Lista o "Ninguno"]*

## Decisión Final
*Toma en cuenta la "Decisión Preliminar" de spec-review y tu propio análisis.*
**DICTAMEN:** APROBADO | RECHAZADO
**Justificación:** *Breve explicación si es rechazado.*
```

Al terminar, reporta al usuario únicamente:
`done → .agents/docs/progress/review/review_{feature}.md`

**Score:** `max(0, 10 - (cant_P1 * 2 + cant_P2 * 0.5))`

**Estados:**
- **APROBADO** — sin P1, P2 menores o ninguno.
- **APROBADO CON OBSERVACIONES** — sin P1, hay P2 que vale la pena resolver.
- **BLOQUEADO** — hay al menos un P1. No mergear hasta resolverlo.

---

## Reglas

- Leer el diff completo antes de emitir cualquier finding.
- No flaggear algo ya resuelto en el propio diff.
- Solo flaggear problemas reales. Si algo está bien, no mencionarlo.
- Cada finding debe citar la línea que lo motiva. Sin cita = confianza ≤ 4.
- La categoría "Completitud de enums" requiere leer código fuera del diff — hacerlo siempre que haya enums modificados.
- No hacer commit, push ni PR — eso no es responsabilidad de esta skill.
- Si el diff es muy grande (+500 líneas), avisar al usuario y preguntar si quiere focalizarse en algún scope específico primero.
