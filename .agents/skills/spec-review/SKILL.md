---
name: spec-review
description: "Paso 1 de la revisión. Valida el cumplimiento del negocio cruzando la implementación con los requerimientos Gherkin, y ejecuta la suite de tests en la terminal para asegurar que no hay regresiones."
---

# Spec Review

Eres responsable de validar que la feature cumple estrictamente con las reglas de negocio definidas.

## Entradas Esperadas

Antes de empezar, debes haber leído:
1. `.agents/docs/specs/{feature}/requirements.md` (Tu fuente de verdad).
2. `.agents/docs/specs/{feature}/tasks.md` (Deben estar todas marcadas `[x]`).
3. `.agents/docs/progress/impl/impl_{feature}.md`.

## Paso 1: Verificación de Estado
- Verifica que el implementer marcó todas las tareas con `[x]` en `tasks.md`. Si falta alguna, es rechazo automático.
- Verifica que el estado en `impl_{feature}.md` sea "COMPLETO".

## Paso 2: Ejecución Real de Tests (Regression Check)
- Inspecciona el entorno para determinar el comando de testing (ej. `npm test`, `pytest`).
- **Ejecuta el comando en la terminal.**
- Si algún test falla, es rechazo automático (Regression bug). 

## Paso 3: Cobertura Gherkin
- Lee el código fuente de los tests.
- Por cada criterio de aceptación (Scenario Gherkin) en `requirements.md`, debes identificar el archivo y la línea exacta del test que lo valida.
- Si un criterio de aceptación no tiene un test verificable, es rechazo automático.

## Salida Esperada

Crea el archivo `.agents/docs/progress/review/review_{feature}.md` (si existe, sobrescríbelo para este nuevo reporte) con el siguiente formato:

```markdown
# Review: {feature}

## 1. Validación de Tests en Terminal
*Salida de la ejecución de tests (¿Pasaron todos?)*

## 2. Cobertura de Criterios de Aceptación
| Scenario | Test que lo cubre | Estado |
|---|---|---|
| [nombre del scenario] | [archivo:línea] | ✅ / ❌ |

## Findings de Negocio
*[Si hay criterios sin cubrir o tasks sin marcar]*

## Decisión Preliminar (Negocio)
APROBADO | RECHAZADO — [motivo]
```

Al terminar, no reportes "done" al usuario todavía. Prepárate para el siguiente paso (`/code-review`).
