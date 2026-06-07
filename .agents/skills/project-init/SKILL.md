---
name: project-init
description: "Usar SIEMPRE al comenzar a trabajar en un proyecto, sea nuevo o ya existente. Esta skill inicializa la estructura de trabajo en .agents/, copia las skills core, permite seleccionar skills opcionales, y llama automáticamente a architecture-builder si no hay contexto de arquitectura. Es el punto de entrada único del workflow."
---

# Project Init — Inicializador del workflow

Prepara el entorno de trabajo para cualquier proyecto. Crea la estructura `.agents/`, instala las skills necesarias y establece el contexto de arquitectura.



## Paso 1 — Verificar estado actual

Revisar si ya existe la carpeta `.agents/` en la raíz del proyecto.

- Si existe → **parar**. Avisar:

> “Este proyecto ya fue inicializado. `.agents/` existe. Si querés reinicializarlo, eliminá la carpeta manualmente y volvé a correr esta skill.”
  
  No continuar.
- Si no existe → continuar al Paso 2.

-----

## Paso 2 — Crear estructura base

Ejecutar en la terminal desde la raíz del proyecto:

```bash
mkdir -p .agents/skills
mkdir -p .agents/docs
```

Confirmar que las carpetas fueron creadas antes de continuar.

-----

## Paso 3 — Copiar skills core

Copiar todas las skills de la carpeta global de skills core al proyecto:

```bash
cp [RUTA_SKILLS_DIEGO]/skills-core/* .agents/skills/
```

> ⚠️ `[RUTA_SKILLS_DIEGO]` debe ser reemplazada con la ruta absoluta real a tu carpeta `skills-diego` en esta máquina. Modificar esta skill con la ruta correcta antes de usarla.

Confirmar que los archivos fueron copiados correctamente listando `.agents/skills/`:

```bash
ls .agents/skills/
```

-----

## Paso 4 — Seleccionar skills opcionales

Listar las skills disponibles en `skills-diego/` que no estén en `skills-core/`:

```bash
ls [RUTA_SKILLS_DIEGO]/ --ignore=skills-core
```

Presentar al usuario la lista de skills encontradas y preguntar:

> “Estas son las skills opcionales disponibles:
> [lista de skills encontradas]
> 
> ¿Cuáles querés incluir en este proyecto? Podés decirme los nombres o decir ‘ninguna’.”

Esperar respuesta. Para cada skill seleccionada, copiarla:

```bash
cp [RUTA_SKILLS_DIEGO]/[nombre-skill] .agents/skills/
```

Si el usuario dice “ninguna” → continuar sin copiar nada adicional.

-----

## Paso 5 — Llamar a architecture-builder

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
> **Estructura creada:**
> 
> ```
> .agents/
> ├── skills/
> │   ├── [lista de skills instaladas]
> └── docs/
>     └── architecture.md  ← [generado ahora / ya existía]
> ```
> 
> **Skills instaladas:** [lista]
> **Contexto de arquitectura:** [generado / cargado]”

-----

## Reglas de esta skill

- Si `.agents/` ya existe, no hacer nada. No sobreescribir ni modificar.
- No continuar al Paso 3 si el Paso 2 falló.
- No continuar al Paso 5 si los pasos anteriores no se completaron correctamente.
- La ruta `[RUTA_SKILLS_DIEGO]` debe estar configurada antes de usar esta skill. Si no lo está, advertir al usuario y no ejecutar ningún comando.
- Todo el trabajo de arquitectura lo hace `architecture-builder`. Esta skill no genera ni modifica `architecture.md` directamente.