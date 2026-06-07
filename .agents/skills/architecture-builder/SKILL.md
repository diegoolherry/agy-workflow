---
name: architecture-builder
description: "Usar SIEMPRE al iniciar un proyecto nuevo o cuando no existe docs/architecture.md. También usar cuando task-spec detecta que falta el contexto de arquitectura. Esta skill tiene dos caminos: si el proyecto es nuevo, entrevista al usuario con preguntas adaptativas; si el proyecto ya tiene código, analiza los archivos existentes, extrae lo que puede inferir y solo pregunta lo que no puede deducir del código. Evalúa la complejidad del proyecto y decide si generar un solo archivo o un archivo principal más archivos de dominio en docs/architecture/. Agrega docs/ al .gitignore automáticamente."
---

# Architecture Builder

Genera el contexto de arquitectura para cualquier proyecto. Funciona tanto en proyectos nuevos como en proyectos ya iniciados.


## Antes de empezar

**1. Verificar si `docs/architecture.md` ya existe.**

- Si existe → **parar**. Avisar: “Ya existe `docs/architecture.md`. Si querés actualizarlo, modificalo manualmente.” No continuar.
- Si no existe → continuar.

**2. Verificar si `docs/` está en `.gitignore`.**

- Si no está → agregarlo automáticamente, sin preguntar.
- Si ya está → continuar.

-----

## Detección del estado del proyecto

Analizar si el proyecto tiene código existente:

- Si no hay nada (carpeta vacía o solo README) → **camino A: proyecto nuevo**.
- Si hay código → **camino B: proyecto existente**.

-----

## Camino A — Proyecto nuevo

Entrevista completa en cuatro rondas. Cada ronda espera la respuesta del usuario antes de continuar. Las preguntas se adaptan según lo respondido — no hacer preguntas irrelevantes para el tipo de proyecto.

Hacer **todas las preguntas necesarias**. No cortar por cantidad.

### Ronda 1 — Proyecto base

- ¿Cuál es el nombre del proyecto y su objetivo central? (una oración)
- ¿Qué tipo de proyecto es? (web app MVC, API REST, CLI, mobile, biblioteca, otro)
- ¿Cuál es el stack completo? (lenguaje, framework, base de datos, ORM, deploy)

### Ronda 2 — Estructura y capas

Adaptar según el tipo respondido en Ronda 1.

- ¿Cuál es la estructura de carpetas principal?
- ¿Qué capas tiene el sistema? Para cada una: ¿qué hace y qué NO hace?
- ¿Hay autenticación? Si sí: ¿cómo funciona?
- ¿Las bajas de registros son lógicas o físicas?

### Ronda 3 — Reglas de negocio ⚠️

Esta es la ronda más importante.

- ¿Cuáles son las entidades principales y cómo se relacionan?
- ¿Cuáles son las reglas de negocio más críticas? (las que si se violan dejan el sistema en estado inválido)
- ¿Hay distinciones conceptuales que no deben colapsarse?
- ¿Hay flujos con pasos obligatorios en orden específico?
- ¿Qué restricciones del dominio debe respetar el modelo siempre, sin excepción?

### Ronda 4 — Convenciones y prohibiciones

- ¿Hay convenciones de nombrado específicas? (idioma, sufijos, prefijos, patrones)
- ¿Qué cosas el modelo NUNCA debe hacer en este proyecto?
- ¿Hay decisiones técnicas ya tomadas que no están en discusión?
- ¿Algo más que el modelo deba saber para no tomar decisiones incorrectas?

-----

## Camino B — Proyecto existente

### Paso 1 — Análisis del proyecto

Leer y analizar los archivos existentes. Extraer todo lo inferible:

- **Stack:** archivos de configuración, manifests de dependencias, archivos de proyecto
- **Estructura:** árbol de carpetas y archivos
- **Capas:** organización del código, namespaces, módulos
- **Entidades y relaciones:** modelos, esquemas, clases de dominio
- **Convenciones:** patrones de nombrado observados en el código
- **Configuraciones técnicas:** autenticación, migraciones, ORM, etc.

### Paso 2 — Presentar hallazgos

Presentar al usuario un resumen de lo inferido y esperar confirmación o correcciones antes de continuar.

### Paso 3 — Preguntar solo lo que falta

**Siempre preguntar — esto nunca se infiere del código:**

- ¿Cuáles son las reglas de negocio más críticas?
- ¿Hay distinciones conceptuales que no deben colapsarse?
- ¿Hay flujos con pasos obligatorios en orden específico?
- ¿Qué cosas el modelo NUNCA debe hacer en este proyecto?

**Preguntar solo si no pudo inferirse del código:**

- Stack, capas, convenciones, bajas lógicas vs físicas, autenticación.

-----

## Evaluación de complejidad

Con toda la información recopilada, antes de generar los archivos, evaluar la complejidad del proyecto:

**Proyecto simple → un solo archivo**
Se considera simple cuando:

- Tiene un único dominio de negocio sin subdominios claramente separables.
- Las reglas de negocio son pocas y aplican de forma uniforme a todo el proyecto.
- La sección de reglas de negocio no superaría ~40 líneas.

→ Generar únicamente `docs/architecture.md` con todo el contenido.

**Proyecto complejo → archivo principal + carpeta de dominios**
Se considera complejo cuando:

- Tiene múltiples dominios de negocio con reglas propias que no se aplican al resto.
- Cargar las reglas de un dominio para una tarea de otro dominio sería ruido innecesario.
- La sección de reglas de negocio superaría ~40 líneas en un solo archivo.

→ Generar `docs/architecture.md` con el contexto global y un archivo por dominio en `docs/architecture/`.

-----

## Generación de archivos

### Caso simple — solo `docs/architecture.md`

```markdown
# Architecture — [Nombre del proyecto]

## 1. Proyecto
**Objetivo:** [una oración]
**Tipo:** [tipo de proyecto]

## 2. Stack tecnológico
| Componente | Tecnología |
|---|---|

## 3. Estructura del proyecto
[árbol de carpetas con una línea de descripción por carpeta]

## 4. Capas y responsabilidades
### [Nombre de capa]
**Hace:** ...
**No hace:** ...
[repetir por cada capa]

## 5. Reglas de negocio ⚠️
[lista numerada, ordenada de más a menos crítica]

## 6. Convenciones de nombrado
| Elemento | Patrón | Ejemplo |
|---|---|---|

## 7. Reglas técnicas
[autenticación, bajas, testing, deploy, decisiones técnicas fijas]

## 8. Prohibiciones explícitas
[lista de lo que el modelo NUNCA debe hacer en este proyecto]
```

-----

### Caso complejo — `docs/architecture.md` + `docs/architecture/`

**`docs/architecture.md`** contiene solo el contexto global:

```markdown
# Architecture — [Nombre del proyecto]

## 1. Proyecto
**Objetivo:** [una oración]
**Tipo:** [tipo de proyecto]

## 2. Stack tecnológico
| Componente | Tecnología |
|---|---|

## 3. Estructura del proyecto
[árbol de carpetas con una línea de descripción por carpeta]

## 4. Capas y responsabilidades
### [Nombre de capa]
**Hace:** ...
**No hace:** ...
[repetir por cada capa]

## 5. Dominios de negocio
[lista de dominios con una línea de descripción cada uno]
[para reglas detalladas de cada dominio, ver docs/architecture/]

## 6. Convenciones de nombrado
| Elemento | Patrón | Ejemplo |
|---|---|---|

## 7. Reglas técnicas
[autenticación, bajas, testing, deploy, decisiones técnicas fijas]

## 8. Prohibiciones explícitas
[lista de lo que el modelo NUNCA debe hacer en este proyecto]
```

**`docs/architecture/[nombre-dominio].md`** por cada dominio, con este formato:

```markdown
# Dominio: [Nombre del dominio]

## Entidades principales
[entidades de este dominio y cómo se relacionan entre sí]

## Reglas de negocio ⚠️
[lista numerada, ordenada de más a menos crítica, específicas de este dominio]

## Restricciones
[lo que nunca debe hacerse dentro de este dominio]
```

Crear todos los archivos directamente. El usuario puede modificarlos después.

-----

## Reglas de esta skill

- No inventar nada. Si algo es ambiguo, preguntar antes de asumir.
- Las reglas de negocio son la parte más importante. No resumirlas ni comprimirlas para que queden “prolijas” — que queden completas.
- El contexto generado tiene que ser suficiente para que el modelo, sin conversación previa, entienda el proyecto y no tome decisiones incorrectas.
- En el Camino B, separar claramente qué fue inferido del código y qué fue respondido por el usuario.
- Si el proyecto es complejo, indicar al finalizar qué archivos de dominio se crearon y qué contiene cada uno.