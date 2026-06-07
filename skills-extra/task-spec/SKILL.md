---
name: task-spec
description: "Aplicar SIEMPRE antes de implementar cualquier tarea de desarrollo: features, bugfixes, refactors, migraciones, o cualquier cambio al código. Usar cuando el usuario dice 'implementá', 'creá', 'agregá', 'arreglá', 'refactorizá', 'mové', o cualquier variante que implique escribir o modificar código. Esta skill verifica que el spec esté completo antes de escribir una línea. Sin spec completo, no se implementa nada."
---

# Task Spec — Validación pre-implementación

Verifica que una tarea esté completamente especificada antes de implementarla. El objetivo es eliminar los huecos que el modelo llenaría con suposiciones silenciosas.

## Antes de empezar

**1. Verificar si `docs/architecture.md` existe.**

- Si no existe → **parar**. Decir: “No hay contexto de arquitectura para este proyecto. Ejecutá `architecture-builder` primero y después volvemos con esta tarea.”
- Si existe → leerlo. Es el contexto global obligatorio para toda tarea.

**2. Verificar si existe la carpeta `docs/architecture/`.**

- Si no existe → continuar solo con `docs/architecture.md`.
- Si existe → identificar a qué dominio pertenece la tarea y cargar el archivo correspondiente de `docs/architecture/`. Si la tarea toca múltiples dominios, cargar todos los archivos relevantes.

Todo lo que se implemente debe respetar tanto `docs/architecture.md` como el archivo de dominio correspondiente.

-----

## Detección del tipo de tarea

Antes de preguntar sobre el spec, identificar qué tipo de tarea es. Si no queda claro del mensaje del usuario, preguntar:

> “¿Qué tipo de tarea es esta?”
> 
> - **Feature** — agrega funcionalidad nueva al sistema
> - **Bugfix** — corrige un comportamiento incorrecto existente
> - **Refactor** — cambia la estructura interna sin cambiar el comportamiento
> - **Otro** — describir

El tipo determina qué preguntas se hacen a continuación.

-----

## Preguntas por tipo de tarea

Hacer **todas las preguntas necesarias**. No cortar por cantidad. Agrupar preguntas relacionadas en un mismo mensaje y esperar la respuesta completa antes de continuar con el siguiente grupo. Si una respuesta genera nuevas preguntas, hacerlas antes de avanzar.

-----

### Feature nueva

**Grupo 1 — Camino feliz**

- ¿Qué debe hacer exactamente esta feature en el caso normal?
- ¿Cuál es el input esperado y cuál es el output o efecto esperado?

**Grupo 2 — No-comportamientos**

- ¿Qué NO debe hacer esta feature?
- ¿Qué casos que podrían parecer relacionados quedan explícitamente fuera?

**Grupo 3 — Edge cases**
Por cada uno que aplique, preguntar qué debe pasar:

- ¿Qué pasa si el input es nulo, vacío o con formato incorrecto?
- ¿Qué pasa si el registro que se busca no existe?
- ¿Qué pasa si ya existe un registro que entra en conflicto?
- ¿Qué pasa si la operación falla a mitad?
- ¿Hay otros casos límite específicos del dominio?

**Grupo 4 — Scope**

- ¿Qué capas toca esta tarea? (¿solo service? ¿controller también? ¿vista?)
- ¿Qué archivos o módulos están explícitamente fuera de scope aunque parezcan relacionados?
- ¿Hay alguna funcionalidad existente que no debe romperse con este cambio?

-----

### Bugfix

**Grupo 1 — El problema**

- ¿Cuál es el comportamiento actual (incorrecto)?
- ¿Cuál debería ser el comportamiento correcto?
- ¿En qué condiciones se reproduce el bug?

**Grupo 2 — Causa y restricciones**

- ¿Hay hipótesis sobre la causa raíz?
- ¿Qué partes del sistema NO deben tocarse al hacer el fix?
- ¿Hay comportamientos relacionados que deben seguir funcionando igual?

**Grupo 3 — Scope**

- ¿Qué archivos o capas están en scope para el fix?
- ¿Qué queda explícitamente fuera?

-----

### Refactor

**Grupo 1 — Qué cambia**

- ¿Qué cambia exactamente? (estructura, nombrado, organización, extracción de lógica)
- ¿Cuál es el motivo del refactor?

**Grupo 2 — Qué no puede cambiar**

- ¿Qué comportamiento debe ser idéntico antes y después del refactor?
- ¿Hay contratos externos (endpoints, interfaces, eventos) que no pueden cambiar?

**Grupo 3 — Scope**

- ¿Qué archivos o módulos están en scope?
- ¿Qué queda explícitamente fuera aunque parezca relacionado?

-----

## Resumen y confirmación

Con todas las respuestas, presentar un resumen estructurado de lo entendido antes de implementar:

> “Antes de arrancar, esto es lo que entendí:
> 
> **Tipo:** [feature / bugfix / refactor]
> **Qué hace:** [descripción del camino feliz]
> **Qué no hace:** [no-comportamientos explícitos]
> **Edge cases:** [lista de qué pasa en cada caso límite]
> **Scope:** [capas y archivos que toca / que no toca]
> 
> ¿Está correcto o hay algo que corregir?”

Esperar confirmación explícita del usuario. No implementar hasta recibirla.

-----

## Reglas de esta skill

- Sin spec completo, no se implementa nada.
- Si una respuesta del usuario genera nueva ambigüedad, preguntar antes de avanzar. No asumir.
- El resumen debe ser suficientemente específico para que el usuario detecte si hubo un malentendido. No resumir en generalidades.
- Todo lo que se implemente debe respetar `docs/architecture.md` y el archivo de dominio correspondiente si existe. Si hay contradicción entre el spec de la tarea y la arquitectura, señalarlo antes de implementar.