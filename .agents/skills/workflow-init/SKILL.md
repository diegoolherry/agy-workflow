---
name: workflow-init
description: "Usar SIEMPRE al comenzar a trabajar en un proyecto, sea nuevo o ya existente. Esta skill inicializa la estructura de trabajo en .agents/, creando las carpetas base (skills, agents, docs, progress), y llama automáticamente a architecture-builder si no hay contexto de arquitectura. Es el punto de entrada único del workflow."
---

# WorkFlow Init — Inicializador del workflow

Prepara el entorno de trabajo para cualquier proyecto. Crea la estructura `.agents/` y establece el contexto de arquitectura.



## Paso 1 — Verificar estado actual

Revisar si ya existe la carpeta `.agents/` en la raíz del proyecto.

- Si existe → **parar**. Avisar:

> “Este proyecto ya fue inicializado. `.agents/` existe. Si querés reinicializarlo, eliminá la carpeta manualmente y volvé a correr esta skill.”
  
  No continuar.
- Si no existe → continuar al Paso 2.

-----

## Paso 2 — Crear estructura base

Ejecutar en la terminal desde la raíz del proyecto para crear todas las carpetas necesarias:

```bash
mkdir -p .agents/skills
mkdir -p .agents/agents
mkdir -p .agents/docs/logs
mkdir -p .agents/docs/specs
mkdir -p .agents/docs/progress/researchs
mkdir -p .agents/docs/progress/impl
mkdir -p .agents/docs/progress/review
mkdir -p .agents/docs/architecture/decisions
```

Crear también los archivos de bitácora iniciales en `progress/`:

```bash
touch .agents/docs/progress/current.md
touch .agents/docs/progress/history.md
```

Confirmar que las carpetas y archivos fueron creados antes de continuar.

-----

## Paso 3 — Llamar a architecture-builder

Verificar si existe `.agents/docs/architecture.md`:

```bash
ls .agents/docs/
```

**Si no existe** → ejecutar la skill `architecture-builder` ahora.

> “La estructura está lista. No hay contexto de arquitectura todavía — arrancamos con `architecture-builder`.”

Ejecutar `architecture-builder` con el path de docs apuntando a `.agents/docs/`.

**Si existe** → cargar el archivo y avisar:

> “La estructura está lista. Ya hay contexto de arquitectura — el proyecto está listo para trabajar.”

Leer `architecture-builder` y si existe la carpeta `.agents/docs/architecture/`, cargar también los archivos de dominio correspondientes.

-----

## Resumen al finalizar

Al terminar la inicialización, mostrar:

> “✅ Proyecto inicializado.
> 
> **Estructura base creada:**
> 
> ```
> .agents/
> ├── agents/
> ├── skills/
> └── docs/
>     ├── logs/
>     ├── specs/
>     ├── progress/
>     │   ├── researchs/
>     │   ├── impl/
>     │   ├── review/
>     │   ├── current.md
>     │   └── history.md
>     ├── architecture/
>     │    └── decisions/   ← ADRs de decisiones técnicas y de seguridad
>     └── architecture.md  ← [generado ahora / ya existía]
> ```
> 
> **Contexto de arquitectura:** [generado / cargado]”

-----

## Reglas de esta skill

- Si `.agents/` ya existe, no hacer nada. No sobreescribir ni modificar.
- No continuar al Paso 3 si el Paso 2 falló.
- Todo el trabajo de arquitectura lo hace `architecture-builder`. Esta skill no genera ni modifica `architecture.md` directamente.
