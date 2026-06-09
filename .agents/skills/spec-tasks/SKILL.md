---
name: spec-tasks
description: "Escribe el desglose de tareas de implementación de una feature (tasks.md). Úsala como el Paso 3 final en la creación de specs."
---

# Spec Tasks

Eres responsable de crear una checklist concreta y ejecutable de tareas para que el implementador pueda hacer TDD.

## Reglas Principales

- **Atomicidad:** Cada task debe ser implementable de forma independiente y verificable con un test unitario.
- **Accionabilidad:** No usar verbos abstractos. Especificar qué archivo crear o modificar y qué función agregar.
- **Completitud:** Las tasks en conjunto deben satisfacer todo el `design.md` y los `requirements.md`.

## Entradas Esperadas

Antes de ejecutar esta skill, debes haber generado y procesado:
1. `.agents/docs/specs/{feature}/requirements.md` (Paso 1).
2. `.agents/docs/specs/{feature}/design.md` (Paso 2).

## Salida Esperada

Escribe el archivo en `.agents/docs/specs/{feature}/tasks.md` siguiendo ESTRICTAMENTE este formato:

```markdown
# Tasks: {feature}

*Lista ordenada de tasks de implementación.*

- [ ] T1: [descripción concreta de qué crear o modificar]
- [ ] T2: [descripción concreta]
- [ ] T3: [descripción concreta]
```

*Nota: No asumir nada que no esté en el research o en architecture.md. Verificar si los ADRs en `.agents/docs/architecture/decisions/` imponen restricciones sobre la forma de implementar alguna task. Si algo es ambiguo, dejarlo marcado con [PENDIENTE: descripción de la duda].*

Al terminar, reporta `done → tasks.md`.
