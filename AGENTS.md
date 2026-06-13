# AGENTS.md — Mapa de navegación para agentes de IA

> Este archivo es el **punto de entrada** para cualquier agente que trabaje en este repositorio. NO es una biblia de reglas: es un **mapa**. Lee solo lo que necesites cuando lo necesites (divulgación progresiva).

---

## 1. Antes de empezar (obligatorio)

1. Ejecuta `/workflow-init` si la carpeta `.agents/` no existe en la raíz del proyecto. Si el comando falla, **para** y resuelve el entorno antes de intentar tocar código.
2. Asegúrate de haber leído `GEMINI.md` para entender tus restricciones de sistema (`/leader` y `/caveman`).
3. Lee `.agents/docs/progress/current.md` para entender en qué estado quedó la última sesión.
4. Lee `feature_list.json`. Toda feature nueva pasa por el flujo SDD.
5. Lee `.agents/docs/architecture.md` y revisa la carpeta `.agents/docs/architecture/decisions/` antes de diseñar o implementar código nuevo.

## 2. Mapa del repositorio

| Archivo / carpeta            | Qué contiene                                                                | Cuándo leerlo |
|------------------------------|-----------------------------------------------------------------------------|---------------|
| `GEMINI.md`                  | Contexto base, rol de `/leader` y restricciones de `/caveman`               | Al iniciar la sesión (automático) |
| `feature_list.json`          | Lista de tareas con estado (`pending` / `in_progress` / `done`)             | Siempre, al empezar |
| `.agents/docs/progress/current.md`| Estado de la sesión actual                                                  | Siempre, al empezar |
| `.agents/docs/progress/history.md`| Bitácora append-only de features completadas                                | Si necesitas contexto histórico |
| `.agents/docs/specs/<feature>/`| `requirements.md` + `design.md` + `tasks.md`                                | Antes de implementar código |
| `.agents/docs/architecture.md` | Decisiones estructurales, dependencias y convenciones base                  | Antes de implementar |
| `.agents/docs/architecture/decisions/` | ADRs (Architecture Decision Records) inmutables                             | Antes de escribir specs o código |
| `.agents/docs/logs/`         | Registros automáticos de finalización de tareas                             | Al buscar historial de errores previos |
| `.agents/skills/`            | Archivos de configuración de los slash commands (`/leader`, etc.)           | Para entender tus comandos disponibles |
| `.agents/agents/`            | Configuración de los subagentes (`implementer`, `spec_author`, etc.)        | Para entender cómo actúan las skills |

## 3. Reglas duras (no negociables)

- **Una sola feature a la vez.** No inicies trabajo nuevo si hay una feature marcada como `in_progress`.
- **No saltes la fase de spec.** Toda feature pasa obligatoriamente por `/spec-requirements`, `/spec-design` y `/spec-tasks` antes de tocar código fuente.
- **No saltes la puerta de aprobación humana.** El flujo se detiene cuando la spec está lista, esperando que el usuario (humano) la apruebe.
- **Regla del teléfono descompuesto:** Todos los subagentes devuelven rutas a archivos guardados en `.agents/docs/`, nunca devuelven código ni resúmenes extensos por chat.
- **Documenta lo que haces** en `current.md` mientras trabajas, no al final de la sesión.

## 4. Flujo de trabajo (SDD)

```text
pending → [/feature-research] → [/spec-requirements → /spec-design → /spec-tasks] → ⏸ HUMANO → in_progress → [/tdd-implement] → [/spec-review → /code-review] → (opcional: /security-audit) → done
```

1. El orquestador (`/leader`) detecta la feature `pending`.
2. El `researcher` ejecuta la skill `/feature-research`.
3. El `spec_author` ejecuta la secuencia de specs: `/spec-requirements`, luego `/spec-design`, y finalmente `/spec-tasks`.
4. **Pausa.** El humano lee los archivos de spec y aprueba (o pide cambios).
5. Una vez aprobado, el leader cambia el status a `in_progress` y el `implementer` ejecuta `/tdd-implement`.
6. El `reviewer` verifica trazabilidad y calidad ejecutando `/spec-review` seguido de `/code-review`.
7. Si la feature toca componentes sensibles (auth, DB, concurrencia), el `security_auditor` ejecuta `/security-audit`.

## 5. Cierre de sesión (lifecycle)

Antes de terminar:

1. El sistema debe tener sus tests en verde.
2. Invoca `/ia-logger`. La herramienta se encarga de redactar el log automáticamente, no debes escribir tú los resúmenes a mano.
3. Marca el estado como `"done"` en `feature_list.json`.
4. Mueve el resumen de `.agents/docs/progress/current.md` al final de `.agents/docs/progress/history.md`.
5. Vacía `current.md` dejando solo la plantilla inicial y limpia.

## 6. Si te bloqueas

- Si un subagente falla repetidamente en su tarea (por ejemplo, `/tdd-implement` falla 3 veces seguidas en un test), **no inventes workarounds a ciegas ni sigas loopeando**. Frena de inmediato y ejecuta `/diagnose`.
- Documenta siempre el bloqueo en `.agents/docs/progress/current.md` y pide intervención al humano si la herramienta de diagnóstico no logra resolverlo de forma segura.
