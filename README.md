# DiegoAI-Stack V2.0

Sistema de trabajo agéntico para desarrollo de software con Antigravity CLI. Organiza un equipo de agentes especializados — cada uno con un rol fijo — que colaboran para investigar, diseñar, implementar, revisar y auditar features dentro de tu proyecto.

Pensado para desarrolladores que quieren trabajar con IA de forma estructurada: el agente no improvisa, no asume y no mezcla responsabilidades. Vos mantenés el control.

---

## ¿Qué problema resuelve esto?

Cuando empezás a usar IA para programar, todo parece mágico al principio. Le pedís que haga algo, lo hace. Pero a medida que el proyecto crece, empiezan los problemas:

- El modelo asume cosas que no debería asumir
- No recuerda las decisiones de arquitectura que tomaste antes
- Mezcla tareas y rompe cosas que ya funcionaban
- No sabés qué hizo exactamente ni por qué
- Cada sesión empieza desde cero

**DiegoAI-Stack V2.0** resuelve eso con un sistema de roles, protocolos y estado persistente en disco. El modelo sigue instrucciones concretas — no adivina.

---

## Filosofía

Estos son los principios que guían todo el sistema. Cada skill y cada agente los respeta.

**Una feature a la vez.**
El sistema rechaza abrir trabajo nuevo si hay algo `in_progress`. Foco total hasta terminar.

**Estado en disco, no en chat.**
Todo resultado de un subagente se escribe en un archivo. El chat no transporta código ni specs completas. Si la sesión se corta, no perdés nada.

**Aprobación humana en los puntos críticos.**
Antes de implementar, el humano aprueba la spec. El modelo no avanza solo.

**Roles separados y no negociables.**
El que investiga no escribe specs. El que escribe specs no implementa. El que implementa no revisa. El que revisa no edita. El leader no toca código.

**Memoria inmutable (ADRs).**
Cada decisión técnica o de seguridad relevante se registra como un *Architecture Decision Record* en `.agents/docs/architecture/decisions/`. Las decisiones no se pierden entre sesiones y todos los agentes las respetan.

---

## Requisitos

- [Antigravity CLI](https://antigravity.google) instalado y configurado
- Git
- Cualquier proyecto de software (el stack es agnóstico al lenguaje)

---

## Instalación

**1. Clonar este repositorio:**

```bash
git clone https://github.com/diegoolherry/agy-workflow.git
```

**2. Copiar los agentes a tu entorno de Antigravity:**

```bash
# macOS / Linux
cp -r .agents/agents/* ~/.gemini/antigravity-cli/agents/

# Windows
xcopy .agents\agents\* %USERPROFILE%\.gemini\antigravity-cli\agents\ /E /I
```

**3. Inicializar un proyecto:**

Navegá a la raíz de tu proyecto y ejecutá en Antigravity CLI:

```
/workflow-init
```

Eso crea toda la estructura `.agents/`, genera el contexto de arquitectura y deja el proyecto listo para trabajar.

---

## Estructura del sistema

### El repositorio

```
DiegoAI-Stack/
└── .agents/
    ├── skills/               → skills core (slash commands del sistema)
    │   ├── workflow-init/
    │   ├── architecture-builder/
    │   ├── leader/
    │   ├── feature-research/
    │   ├── spec-requirements/
    │   ├── spec-design/
    │   ├── spec-tasks/
    │   ├── tdd-implement/
    │   ├── spec-review/
    │   ├── code-review/
    │   ├── security-audit/
    │   ├── diagnose/
    │   └── ia-logger/
    └── agents/               → configuración de los subagentes especializados
        ├── researcher/
        │   └── agent.json
        ├── spec_author/
        │   └── agent.json
        ├── implementer/
        │   └── agent.json
        ├── reviewer/
        │   └── agent.json
        └── security_auditor/
            └── agent.json
```

### Tu proyecto (después de `/workflow-init`)

```
tu-proyecto/
├── feature_list.json          → lista de features con su estado
└── .agents/
    ├── skills/
    ├── agents/
    └── docs/                  → documentación generada (va en .gitignore)
        ├── architecture.md    → fuente de verdad del sistema
        ├── architecture/
        │   └── decisions/     → ADRs: historial inmutable de decisiones
        ├── specs/
        │   └── {feature}/
        │       ├── requirements.md
        │       ├── design.md
        │       └── tasks.md
        ├── logs/
        │   └── {nn}-{descripcion}.md
        └── progress/
            ├── current.md
            ├── history.md
            ├── researchs/
            │   └── research_{feature}.md
            ├── impl/
            │   └── impl_{feature}.md
            └── review/
                ├── review_{feature}.md
                └── security_{feature}.md
```

---

## Las skills — los comandos del sistema

Las skills son archivos `.md` que Antigravity CLI convierte en slash commands. Escribís `/nombre-skill` y el agente ejecuta el protocolo definido en ese archivo.

### `/workflow-init`

**Cuándo usarla:** siempre que empezás a trabajar en un proyecto, sea nuevo o ya existente.

**Qué hace:**
1. Crea la estructura completa de `.agents/` (skills, agents, docs, progress, specs, logs, decisions)
2. Llama automáticamente a `/architecture-builder` si no hay contexto de arquitectura

---

### `/architecture-builder`

**Cuándo usarla:** al iniciar un proyecto o cuando querés regenerar el contexto.

**Qué hace:**

En proyectos nuevos, te hace preguntas sobre el dominio, stack y reglas de negocio. En proyectos existentes, analiza el código y solo pregunta lo que no puede deducir.

Genera el contexto en función de la complejidad:
- Proyecto simple → `architecture.md`
- Proyecto complejo → `architecture.md` + archivos por dominio en `architecture/`

---

### `/leader`

**Cuándo usarla:** para implementar una feature completa de punta a punta.

**Qué hace:** orquesta todos los subagentes siguiendo el flujo SDD. También responde a modos predefinidos:

- `pre-deploy` → auditoría de seguridad completa
- `pre-merge` → code review antes de mergear
- `post-bugfix` → verifica el fix y lo registra

Ver sección "El flujo SDD" para el detalle completo.

---

### `/diagnose`

**Cuándo usarla:** cuando hay un bug, error o comportamiento inesperado.

**Qué hace:** aplica un loop de debugging disciplinado:

1. Construye un feedback loop reproducible (sin loop, no hay hipótesis)
2. Genera 3 a 5 hipótesis falsificables rankeadas
3. Las testea de a una con la regla de 3 strikes
4. Aplica fix de causa raíz + regression test
5. Emite un DEBUG REPORT estructurado

---

### `/security-audit`

**Cuándo usarla:** antes de poner algo en producción, o es invocada automáticamente por el leader cuando una feature toca auth, db o concurrencia.

**Modos de operación:**
- **Modo Feature** — audita solo los archivos modificados por la feature en curso. Aplica OWASP y STRIDE de forma focalizada.
- **Modo Global** — escanea el proyecto entero: historial git, dependencias con CVEs, superficies de ataque completas.

Si detecta un riesgo estructural, emite un **ADR** en `.agents/docs/architecture/decisions/` que todos los agentes respetarán en el futuro.

---

### `/ia-logger`

**Cuándo usarla:** se ejecuta automáticamente al finalizar cada feature. No necesitás invocarla.

**Qué hace:** crea un archivo de log en `.agents/docs/logs/` con: fecha, modelo, qué se hizo, decisión clave tomada, problemas encontrados y cómo contribuyó la IA.

---

## Los agentes del flujo SDD

Los agentes son subagentes especializados que `/leader` coordina. Cada uno tiene un input esperado, un output concreto y no puede salirse de su rol.

### `researcher` — Codebase Researcher

Solo lectura. Nunca modifica código.

Analiza el codebase antes de que se escriba cualquier spec. Lee el contexto de arquitectura, los ADRs vigentes y el código relevante para detectar fricciones, convenciones y dependencias que impactan la nueva feature.

**Output:** `.agents/docs/progress/researchs/research_{feature}.md`

---

### `spec_author` — Spec Author

No escribe código.

Con el research como base, genera los 3 archivos de spec ejecutando sus skills modulares en secuencia:

1. `/spec-requirements` → `requirements.md` con criterios de aceptación en Gherkin
2. `/spec-design` → `design.md` con decisiones técnicas, modelo de datos e interfaces
3. `/spec-tasks` → `tasks.md` con checklist atómico, una task = un test

Si algo es ambiguo, lo marca con `[PENDIENTE: descripción]` en vez de asumir.

**Output:** `.agents/docs/specs/{feature}/`

---

### `implementer` — TDD Implementer

Implementa la spec aprobada siguiendo TDD estricto. El ciclo por cada task es:

1. **RED** — escribe el test. Lo ejecuta en la terminal y verifica que falla.
2. **GREEN** — escribe el código mínimo para que pase. Nada más.
3. **REFACTOR** — limpia lo que acaba de escribir (solo ese archivo). No refactoriza lo que no tocó.
4. **COMMIT** — hace un commit atómico con `feat: [descripción] [T{n}]`.

Si falla 3 veces en una task, invoca `/diagnose`. Si el bloqueo persiste, reporta `BLOQUEADO en T{n}` y para.

**Output:** `.agents/docs/progress/impl/impl_{feature}.md`

---

### `reviewer` — Code Reviewer

Nunca edita código.

Realiza la revisión en dos pasadas:

1. `/spec-review` — verifica que cada criterio Gherkin tiene un test real que lo cubre. Ejecuta la suite completa en la terminal. Un criterio sin test es rechazo automático.
2. `/code-review` — evalúa calidad, arquitectura y coherencia con los ADRs vigentes.

Emite APROBADO o RECHAZADO con findings que incluyen archivo y línea.

**Output:** `.agents/docs/progress/review/review_{feature}.md`

---

### `security_auditor` — Security Auditor

Solo es invocado cuando la feature toca dominios sensibles (auth, db, concurrencia).

Aplica STRIDE y OWASP Top 10 sobre los archivos modificados en Modo Feature. Si encuentra un bug con fix directo, el leader devuelve el trabajo al implementer (Auto-Fix loop). Si descubre un riesgo estructural, emite un ADR y actualiza `architecture.md`.

**Output:** `.agents/docs/progress/review/security_{feature}.md`

---

## El flujo SDD

SDD (Spec-Driven Development) es el flujo completo orquestado por `/leader`.

```
/leader "implementá {feature}"
         │
         ├─ Fase 0 — verificación
         │    Lee feature_list.json
         │    Si hay algo in_progress → para. Una feature a la vez.
         │    Marca la feature como in_progress
         │
         ├─ Fase 1 — research
         │    Agente: researcher
         │    Lee architecture.md, ADRs y código relevante
         │    Output: progress/researchs/research_{feature}.md
         │
         ├─ Fase 2 — spec
         │    Agente: spec_author
         │    Genera: requirements.md → design.md → tasks.md
         │
         ├─ ⏸ PAUSA — aprobación humana obligatoria
         │    El leader te avisa dónde están los archivos
         │    Revisás criterios, diseño y tasks
         │    Sin tu "aprobado", no continúa
         │
         ├─ Fase 3 — implementación (TDD)
         │    Agente: implementer
         │    Ciclo RED → GREEN → REFACTOR → COMMIT por cada task
         │    Si se bloquea → /diagnose → si persiste → escala al usuario
         │    Output: progress/impl/impl_{feature}.md
         │
         ├─ Fase 4 — review
         │    Agente: reviewer
         │    Pasada 1: negocio + tests en terminal
         │    Pasada 2: calidad + ADRs
         │    RECHAZADO → vuelve a Fase 3 con feedback concreto (máx. 2 veces)
         │    Output: progress/review/review_{feature}.md
         │
         ├─ Fase 5 — seguridad (condicional)
         │    Agente: security_auditor (solo si toca auth, db o concurrencia)
         │    Si hay bug crítico con fix directo → vuelve a Fase 3 (1 vez)
         │    Si hay riesgo estructural → emite ADR
         │    Output: progress/review/security_{feature}.md
         │
         └─ Fase 6 — cierre
              feature_list.json → "status": "done"
              /ia-logger registra la tarea en disco
              progress/history.md se actualiza
              Resumen final al usuario
```

---

## El ecosistema ADR

Los *Architecture Decision Records* son la memoria a largo plazo del sistema.

Cuando tomas una decisión técnica importante — o cuando el `security_auditor` descubre una restricción estructural — se crea un archivo en `.agents/docs/architecture/decisions/` con este formato:

```markdown
# ADR-001: Uso de JWT stateless para autenticación

**Fecha:** 09/06/2026
**Estado:** Vigente
**Contexto:** El sistema necesita autenticación sin estado para escalar horizontalmente.
**Decisión:** JWT con expiración corta (15 min) + refresh token. Sin blacklist server-side.
**Consecuencias:** No es posible invalidar tokens antes de su expiración. El logout es client-side.
```

**¿Por qué importa?** Todos los agentes del stack leen la carpeta `decisions/` antes de actuar. Si el `spec_author` intenta diseñar algo que viola un ADR, debe documentar la excepción y justificarla. El `architecture-builder` no puede sobreescribir ADRs existentes. El `code-review` sabe que una violación intencionada de un ADR es más grave que un bug común.

---

## `feature_list.json`

Este archivo vive en la raíz del proyecto y es el registro de todas las features. Lo crea `/workflow-init` vacío y lo mantiene `/leader`.

```json
[
  { "id": "001", "name": "autenticación con JWT", "status": "done" },
  { "id": "002", "name": "registro de usuarios",  "status": "in_progress" },
  { "id": "003", "name": "recuperación de contraseña", "status": "pending" }
]
```

Los estados posibles son `pending`, `in_progress` y `done`. El sistema rechaza tener más de un `in_progress` al mismo tiempo.

---

## Estado en disco

El workflow mantiene dos archivos de estado en `.agents/docs/progress/`:

**`current.md`** — estado de la sesión activa. Se sobreescribe al iniciar cada sesión. Registra qué feature está en curso, en qué fase y qué decisiones se tomaron.

**`history.md`** — bitácora append-only. Nunca se sobreescribe. Cada feature completada agrega una entrada con fecha, resultado y referencias a los archivos generados.

Esto permite retomar el trabajo en cualquier momento sin depender de que el modelo recuerde la conversación anterior.

---

## Adaptación a Claude Code

La lógica del sistema — el flujo SDD, los roles de los agentes, los ADRs — es 100% portable. Lo que cambia son las convenciones de archivo de cada plataforma.

### Paso 1: Renombrar la carpeta raíz

En Antigravity la carpeta es `.agents/`. En Claude Code es `.claude/`. Todo lo que sigue aplica con ese cambio de nombre.

```
.agents/  →  .claude/
```

### Paso 2: Las skills (sin cambios de contenido)

Las skills son el mismo archivo `SKILL.md`. Solo cambia dónde viven:

| Antigravity | Claude Code |
|---|---|
| `.agents/skills/{nombre}/SKILL.md` | `.claude/skills/{nombre}/SKILL.md` |

El contenido del archivo es idéntico. No hay nada que reescribir.

### Paso 3: Convertir los agentes de `.json` a `.md`

Claude Code no usa `agent.json`. Los agentes son archivos `.md` con frontmatter YAML. La conversión es directa:

**Antigravity (`agent.json`):**
```json
{
  "name": "researcher",
  "persona": {
    "instructions": ["Sos un Codebase Researcher...", "Nunca modificás código."]
  },
  "runtime": { "model": "gemini-3.1-pro-high" },
  "capabilities": { "tools": ["fs_read", "fs_write"] }
}
```

**Claude Code (`researcher.md`):**
```markdown
---
name: researcher
description: Analiza el codebase antes de escribir specs. Solo lectura.
tools: Read, Grep, Glob
model: claude-sonnet-4-6
---

Sos un Codebase Researcher. Nunca modificás código.
[resto de las instrucciones del campo persona.instructions]
```

**Tabla de mapeo de campos:**

| Campo en `.json` | Campo en `.md` frontmatter | Nota |
|---|---|---|
| `name` | `name` | Idéntico |
| `description` | `description` | Idéntico |
| `persona.instructions` | Cuerpo del archivo (debajo del frontmatter) | Unir los items del array como párrafos |
| `runtime.model` | `model` | Usar el nombre del modelo de Claude |
| `capabilities.tools` | `tools` | Ver tabla de herramientas abajo |

**Herramientas disponibles en Claude Code y su equivalente:**

| Antigravity | Claude Code |
|---|---|
| `fs_read` | `Read`, `Grep`, `Glob` |
| `fs_write` | `Write`, `Edit` |
| `terminal` | `Bash` |

### Paso 4: Actualizar las rutas dentro de las skills

Las skills del stack tienen rutas hardcodeadas a `.agents/docs/`. Necesitás hacer un find & replace en todos los archivos:

```bash
# macOS / Linux
grep -rl ".agents/docs" .claude/skills/ | xargs sed -i 's|.agents/docs|.claude/docs|g'
grep -rl ".agents/docs" .claude/agents/ | xargs sed -i 's|.agents/docs|.claude/docs|g'

# Windows (PowerShell)
Get-ChildItem -Recurse -Path .claude | ForEach-Object {
  (Get-Content $_.FullName) -replace '\.agents/docs', '.claude/docs' | Set-Content $_.FullName
}
```

### Paso 5: Crear el CLAUDE.md

Claude Code lee automáticamente `CLAUDE.md` en la raíz del proyecto al iniciar cada sesión. Creá este archivo para que el agente cargue el contexto:

```markdown
# Contexto del proyecto

Antes de cualquier acción, leer:
- `.claude/docs/architecture.md`
- `.claude/docs/architecture/decisions/` (ADRs vigentes)

Todo el output generado va a `.claude/docs/`. Nunca al raíz del proyecto.
```

### Paso 6: Orquestación manual

A diferencia de Antigravity (donde `/leader` spawna agentes automáticamente), en Claude Code la orquestación es manual vía menciones `@`:

```
@researcher analizá el codebase para la feature de autenticación
@spec_author armá la spec completa usando el research
@implementer implementá la feature siguiendo TDD
@reviewer revisá la implementación
```

**Lo que NO cambia:** el flujo SDD completo, las fases, la lógica de los ADRs, el formato de todos los archivos de output, el `feature_list.json` y `SKILL.md` de las skills.

---

## Preguntas frecuentes

**¿Puedo usar las skills por separado sin el flujo SDD completo?**
Sí. Las skills funcionan de forma independiente. Podés usar `/diagnose` cuando hay un bug, `/security-audit` antes de un deploy o `/code-review` antes de mergear, sin necesidad de usar `/leader`.

**¿El modelo puede modificar las skills?**
No debería. Las skills son protocolos que el modelo sigue, no código que ejecuta. Si el modelo intenta modificar una skill para facilitarse el trabajo, es una señal de que algo está mal.

**¿Qué pasa si el `reviewer` rechaza dos veces?**
El leader escala al usuario. No sigue loopeando automáticamente — dos rechazos son señal de un problema más profundo que necesita intervención humana.

**¿Por qué `.agents/docs/` no se versiona?**
Porque es documentación generada en el proceso. Lo que se versiona es el código, las skills y los agentes — no los artefactos intermedios del workflow. `/workflow-init` agrega `.agents/docs/` al `.gitignore` automáticamente.

**¿Qué son los ADRs y tengo que crearlos yo?**
No. Los ADRs los crea automáticamente el `security_auditor` cuando detecta una restricción estructural. Vos podés crear uno manualmente también, pero el sistema funciona sin intervención.

---

## Contribuciones

Este es un proyecto personal en evolución. Si encontrás algo que no funciona, querés proponer una mejora o adaptar el workflow a otro entorno, abrí un issue o un PR.

---

## Licencia

MIT
