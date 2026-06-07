# DiegoAI-Stack

Sistema de trabajo agéntico para desarrollo de software con Antigravity CLI. Organiza skills, agentes especializados y documentación generada dentro de cada proyecto, bajo la carpeta `.agents/`.

Pensado para desarrolladores que quieren trabajar con IA de forma estructurada, sin perder el control del código ni depender de que el modelo "adivine" lo que tiene que hacer.

---

## ¿Qué problema resuelve esto?

Cuando empezás a usar IA para programar, todo parece mágico al principio. Le pedís que haga algo, lo hace. Pero a medida que el proyecto crece, empiezan los problemas:

- El modelo asume cosas que no debería asumir
- No recuerda las decisiones que tomaste antes
- Mezcla tareas y rompe cosas que ya funcionaban
- No sabés qué hizo exactamente ni por qué
- Cada sesión empieza desde cero

**DiegoAI-Stack** resuelve eso con un sistema de reglas, roles y estado persistente. El modelo no improvisa — sigue un protocolo. Vos mantenés el control.

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

**Sin suposiciones silenciosas.**
Antes de implementar cualquier tarea, el sistema valida que el spec esté completo. Si algo es ambiguo, pregunta. No asume.

---

## Requisitos

- [Antigravity CLI](https://antigravity.google) instalado y configurado
- Git
- Cualquier proyecto de software (el stack es agnóstico al lenguaje)

---

## Instalación

**1. Clonar este repo en tu máquina:**

```bash
git clone https://github.com/tu-usuario/DiegoAI-Stack.git
```

**2. Configurar tu ruta global.**

Abrí el archivo `.agents/skills/project-init/project-init-SKILL.md` y reemplazá `[RUTA_SKILLS_DIEGO]` con la ruta absoluta donde clonaste este repo.

Por ejemplo:
- Windows: `C:/Users/tu-usuario/DiegoAI-Stack`
- macOS/Linux: `/home/tu-usuario/DiegoAI-Stack`

**3. Copiar los agentes a tu carpeta global de Antigravity (opcional pero recomendado).**

Antigravity CLI permite agentes globales disponibles en todos los proyectos:

```bash
# macOS / Linux
cp -r .agents/agents/* ~/.gemini/antigravity-cli/agents/

# Windows
xcopy .agents\agents\* %USERPROFILE%\.gemini\antigravity-cli\agents\ /E /I
```

**4. Inicializar un proyecto.**

Navegá a la raíz de tu proyecto y ejecutá en Antigravity CLI:

```
/project-init
```

Eso es todo. El sistema se configura solo.

---

## Estructura del sistema

```
DiegoAI-Stack/
├── .agents/
│   ├── skills/           → skills core que van a todos los proyectos
│   │   ├── project-init/
│   │   ├── architecture-builder/
│   │   ├── task-spec/
│   │   ├── ia-logger/
│   │   ├── diagnose/
│   │   ├── leader/
│   │   ├── code-review/
│   │   └── security-audit/
│   └── agents/           → subagentes especializados del flujo SDD
│       ├── implementer-agent.json
│       ├── researcher-agent.json
│       ├── reviewer-agent.json
│       └── spec_author-agent.json
└── skills-extra/         → skills opcionales por temática
    ├── frontend-desing/
    ├── threejs-skills/
    ├── to-prd/
    ├── ui-ux-pro-max-skill/
    └── zoom-out/
```

Una vez inicializado un proyecto, la estructura dentro de él queda así:

```
tu-proyecto/
├── feature_list.json        → listado de features con su estado
└── .agents/
    ├── skills/              → skills activas en este proyecto
    ├── agents/              → agentes activos en este proyecto
    └── docs/                → toda la documentación generada (en .gitignore)
        ├── architecture.md
        ├── architecture/
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
            ├── research_{feature}.md
            ├── impl_{feature}.md
            └── review_{feature}.md
```

---

## Skills — los comandos del sistema

Las skills son archivos `.md` que Antigravity CLI convierte en slash commands. Escribís `/nombre-skill` y el agente ejecuta el protocolo definido en ese archivo.

### Skills core — van a todos los proyectos

---

### `/project-init`

**Cuándo usarla:** siempre que empezás a trabajar en un proyecto, sea nuevo o ya existente.

**Qué hace:**
1. Crea la estructura completa de `.agents/` (skills, agents, docs, logs, progress, specs)
2. Copia todas las skills core al proyecto
3. Copia los agentes del workflow SDD
4. Te pregunta qué skills opcionales querés incluir
5. Genera `feature_list.json` vacío en la raíz del proyecto
6. Llama automáticamente a `/architecture-builder` si no hay contexto de arquitectura

**Cómo usarla:**
```
/project-init
```

---

### `/architecture-builder`

**Cuándo usarla:** al iniciar un proyecto nuevo o cuando no existe `.agents/docs/architecture.md`. `project-init` la llama automáticamente, pero podés invocarla directamente si necesitás actualizar el contexto.

**Qué hace:**

En proyectos nuevos, te entrevista en 4 rondas:
1. **Proyecto base** — nombre, objetivo, tipo y stack
2. **Estructura y capas** — organización de carpetas, qué hace cada capa
3. **Reglas de negocio** — la parte más importante: entidades, relaciones, restricciones del dominio
4. **Convenciones y prohibiciones** — nombrado, decisiones fijas, cosas que el modelo nunca debe hacer

En proyectos existentes, analiza el código, te presenta lo que infirió y solo pregunta lo que no puede deducir.

Genera uno o varios archivos según la complejidad del proyecto:
- Proyecto simple → `architecture.md`
- Proyecto complejo → `architecture.md` + archivos por dominio en `architecture/`

**Por qué es importante:** sin este archivo, el modelo no tiene contexto del proyecto y toma decisiones incorrectas. Es la base de todo lo demás.

---

### `/task-spec`

**Cuándo usarla:** antes de implementar cualquier tarea — feature, bugfix o refactor.

**Qué hace:**
1. Verifica que existe `architecture.md` (si no, para y te manda a `/architecture-builder`)
2. Detecta el tipo de tarea
3. Te hace todas las preguntas necesarias agrupadas:
   - Camino feliz: ¿qué debe hacer exactamente?
   - No-comportamientos: ¿qué NO debe hacer?
   - Edge cases: ¿qué pasa si el input es nulo, si el registro no existe, si hay conflicto?
   - Scope: ¿qué capas toca? ¿qué queda explícitamente fuera?
4. Presenta un resumen de lo entendido y espera tu confirmación antes de implementar

**La regla:** sin spec completo, no se implementa nada.

**Cómo usarla:**
```
/task-spec
```

> **Nota para el flujo SDD:** si usás `/leader`, el `spec_author` ya aplica este mismo protocolo. No hace falta llamar a `/task-spec` por separado.

---

### `/ia-logger`

**Cuándo usarla:** se ejecuta automáticamente al finalizar cada tarea. No necesitás invocarla manualmente.

**Qué hace:** crea un archivo de log en `.agents/docs/logs/` con este contenido:
- Fecha, hora y modelo de IA utilizado
- Tipo de tarea (feature / bugfix / refactor)
- Qué se hizo
- Decisión clave tomada (si aplica)
- Problemas encontrados y cómo se resolvieron (si aplica)
- Cómo contribuyó la IA
- Estado final y pendientes

**Por qué existe:** tener un registro de qué hizo la IA y por qué es valioso para auditar, aprender y retomar trabajo.

---

### `/diagnose`

**Cuándo usarla:** cuando hay un bug, error, comportamiento inesperado o regresión de performance.

**Qué hace:** aplica un loop de debugging disciplinado en 6 fases:

1. **Construir un feedback loop** — antes de hacer cualquier cosa, construir una señal pass/fail reproducible. Si no podés reproducir el bug, no podés arreglarlo.
2. **Reproducir** — verificar que el loop produce exactamente el bug que describiste
3. **Hipotetizar** — chequear tabla de patrones conocidos (race condition, null propagation, state corruption, etc.) y generar 3 a 5 hipótesis falsificables rankeadas
4. **Instrumentar** — testear hipótesis de a una, con la regla de 3 strikes: si 3 hipótesis fallan, para y escala
5. **Fix + regression test** — fix de causa raíz, no del síntoma. Test antes del fix.
6. **Cleanup + post-mortem** — remover logs de debug, documentar causa raíz

Emite un DEBUG REPORT estructurado al cerrar.

---

### `/leader`

**Cuándo usarla:** para implementar una feature completa o ejecutar un workflow de alto nivel.

**Qué hace:** detecta el modo según lo que le escribís y coordina subagentes:

**Modos predefinidos:**
- `pre-deploy` → `/code-review` + `/security-audit` en paralelo → GO / NO-GO
- `pre-merge` → `/code-review` → APROBADO / BLOQUEADO
- `post-bugfix` → verifica el fix + registra en `/ia-logger`
- `sprint-start` → lee `feature_list.json` + historial → resumen de qué atacar primero

**Flujo SDD (para features nuevas):**
```
researcher → spec_author → [aprobación humana] → implementer → reviewer → cierre
```
Ver sección "El flujo SDD" más abajo para el detalle completo.

**Modo dinámico:** si el objetivo no encaja en ningún modo predefinido, el leader razona qué agentes usar, te presenta el plan y espera tu confirmación antes de ejecutar.

**La regla del leader:** nunca toca código directamente. Todo pasa por subagentes.

---

### `/code-review`

**Cuándo usarla:** antes de mergear cualquier cambio.

**Qué hace:**
1. Obtiene el diff contra la rama base
2. Detecta el scope del cambio (auth, db, api, frontend, concurrencia)
3. Aplica categorías de findings:
   - **P1 (críticos, bloquean el merge):** SQL injection, autorización rota, secrets hardcodeados, race conditions, null references
   - **P2 (informativos):** lógica duplicada, manejo de errores genérico, tests incompletos
4. Cada finding cita la línea exacta que lo motiva y tiene un score de confianza del 1 al 10
5. Emite un CODE REVIEW con score numérico y estado APROBADO / APROBADO CON OBSERVACIONES / BLOQUEADO

---

### `/security-audit`

**Cuándo usarla:** antes de poner algo en producción o cuando querés una revisión de seguridad.

**Qué hace:** auditoría en 7 fases:
1. Modelo mental del sistema + detección del stack
2. Mapa de superficie de ataque (endpoints públicos, autenticados, admin)
3. Arqueología de secretos (historial git, archivos .env, credenciales hardcodeadas)
4. Supply chain (dependencias con CVEs conocidos)
5. OWASP Top 10 (las 10 vulnerabilidades web más comunes)
6. STRIDE threat model por componente
7. Clasificación de datos (qué datos maneja el sistema y qué tan sensibles son)

Emite un SECURITY POSTURE REPORT con findings, escala de confianza y estado.

> ⚠️ Esta auditoría es un primer filtro asistido por IA. Para sistemas en producción con datos sensibles, reemplazá con una auditoría profesional de seguridad.

---

## Los agentes del flujo SDD

Los agentes son subagentes especializados que `leader` spawna durante el flujo SDD. Cada uno tiene un rol fijo y no puede salirse de él.

### `researcher` — Codebase Researcher

Solo lectura. Nunca modifica código.

Analiza el codebase antes de que se escriba cualquier spec. Identifica qué módulos toca la feature nueva, qué convenciones existen, qué dependencias son relevantes y qué fricciones puede haber con código existente.

**Output:** `.agents/docs/progress/research_{feature}.md`

**Modelo:** Gemini 3.1 Pro (High) — necesita razonar sobre arquitectura, no solo leer

---

### `spec_author` — Spec Author

No escribe código.

Con el contexto del researcher, escribe los 3 archivos de spec:

- **`requirements.md`** — criterios de aceptación en formato Gherkin. Cubre camino feliz, edge cases y casos de error.
- **`design.md`** — decisiones técnicas, cambios al modelo de datos, interfaces y contratos con firmas concretas
- **`tasks.md`** — checklist ordenado de tasks. Cada task debe ser implementable de forma independiente y tener un test asociado

Si hay ambigüedades, las marca con `[PENDIENTE: descripción]` en vez de asumir.

**Output:** `.agents/docs/specs/{feature}/`

**Modelo:** Claude Sonnet 4.6 (Thinking) — la spec es el artefacto más crítico del workflow

---

### `implementer` — TDD Implementer

Implementa la spec aprobada siguiendo TDD estricto.

El ciclo por cada task es:
1. **RED** — escribir el test que define el comportamiento esperado. Verificar que falla.
2. **GREEN** — escribir el código mínimo que hace pasar el test. Nada más.

Una task a la vez. No avanza a la siguiente sin tener el test en verde. No refactoriza código adyacente.

Si falla 3 veces en una task, invoca `/diagnose`. Si el bloqueo persiste, reporta `BLOQUEADO en T{n}` y para.

**Output:** `.agents/docs/progress/impl_{feature}.md`

**Modelo:** Claude Sonnet 4.6 (Thinking) — genera código real, es donde más importa la calidad

---

### `reviewer` — Code Reviewer

Nunca edita código.

Valida que la implementación cubre cada criterio de aceptación Gherkin de la spec. Por cada scenario, verifica que existe un test que lo cubre. Un criterio sin test es rechazo automático.

Emite APROBADO o RECHAZADO con findings concretos que incluyen archivo y línea.

**Output:** `.agents/docs/progress/review_{feature}.md`

**Modelo:** Gemini 3.5 Flash (High) — tiene criterios explícitos para comparar, no necesita inferir

---

## El flujo SDD

SDD (Spec-Driven Development) es el flujo completo para implementar una feature. Lo orquesta `/leader`.

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
         │    Output: .agents/docs/progress/research_{feature}.md
         │
         ├─ Fase 2 — spec
         │    Agente: spec_author
         │    Lee: research_{feature}.md + architecture.md
         │    Output: specs/{feature}/requirements.md
         │             specs/{feature}/design.md
         │             specs/{feature}/tasks.md
         │
         ├─ ⏸ PAUSA — aprobación humana obligatoria
         │    El leader te muestra dónde están los archivos
         │    Revisás los criterios de aceptación, el diseño y las tasks
         │    Decís "aprobado" o pedís cambios
         │    Sin tu OK, no continúa
         │
         ├─ Fase 3 — implementación (TDD)
         │    Agente: implementer
         │    Ciclo RED → GREEN por cada task
         │    Si se bloquea → /diagnose → si persiste → escala al usuario
         │    Output: .agents/docs/progress/impl_{feature}.md
         │
         ├─ Fase 4 — review
         │    Agente: reviewer
         │    Valida cada criterio Gherkin contra su test
         │    APROBADO → continuar
         │    RECHAZADO → volver a Fase 3 con feedback concreto
         │    Máximo 2 rechazos antes de escalar al usuario
         │
         └─ Fase 5 — cierre
              feature_list.json → "status": "done"
              /ia-logger registra la tarea
              progress/history.md se actualiza
              Resumen final al usuario
```

### SDD vs TDD — no son lo mismo

Es común confundirlos. Son capas distintas del mismo proceso:

- **SDD** opera *antes* de escribir código. Define qué debe hacer el sistema. El output son specs.
- **TDD** opera *durante* la implementación. El `implementer` escribe el test que falla primero, luego el código que lo hace pasar.
- **Gherkin** es el puente: los criterios de aceptación en `requirements.md` están en formato Gherkin, que luego el `reviewer` usa para verificar que cada scenario tiene su test.

El flujo completo es: SDD define qué hacer → TDD define cómo hacerlo → Gherkin verifica que se hizo.

---

## `feature_list.json`

Este archivo vive en la raíz del proyecto y es el registro de todas las features. Lo crea `/project-init` vacío y lo mantiene `/leader`.

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

**`current.md`** — estado de la sesión activa. Se sobreescribe al iniciar cada sesión. Registra qué feature está en curso, en qué fase está, qué agentes están activos y qué decisiones se tomaron.

**`history.md`** — bitácora append-only. Nunca se sobreescribe. Cada feature completada agrega una entrada con fecha, resultado y referencias a los archivos generados.

Esto permite retomar el trabajo en cualquier momento sin depender de que el modelo recuerde la conversación anterior.

---

## Convenciones

- Las skills son archivos `.md` en `.agents/skills/` — se convierten en slash commands en Antigravity CLI
- Los agentes viven en `.agents/agents/` (ej. `implementer-agent.json`)
- Los logs de `ia-logger` usan numeración secuencial de dos dígitos: `01`, `02`, `03`...
- Las specs se organizan por feature: `.agents/docs/specs/{feature}/`
- Todo lo generado por el workflow vive en `.agents/docs/` — nunca en la raíz del proyecto
- `.agents/docs/` va en `.gitignore` — la documentación generada no se versiona. `project-init` lo agrega automáticamente.

---

## Preguntas frecuentes

**¿Puedo usar esto sin `/leader`?**
Sí. Las skills funcionan de forma independiente. Podés usar `/task-spec` antes de implementar, `/diagnose` cuando hay un bug y `/code-review` antes de mergear, sin necesidad de usar el flujo SDD completo.

**¿Qué pasa si quiero usar esto con Claude Code o Cursor?**
El sistema está diseñado para Antigravity CLI. Algunas skills funcionan igual en otros entornos, pero los agentes (`agent.json`) son específicos de Antigravity. La adaptación a otros entornos está en los planes futuros.

**¿El modelo puede modificar las skills?**
No debería. Las skills son protocolos que el modelo sigue, no código que ejecuta. Si el modelo intenta modificar una skill, algo está mal.

**¿Qué pasa si el `reviewer` rechaza dos veces?**
El leader escala al usuario. No sigue loopeando automáticamente — eso sería una señal de que hay un problema más profundo que necesita intervención humana.

**¿Por qué `.agents/docs/` no se versiona?**
Porque es documentación generada. Lo que se versiona es el código y las specs una vez aprobadas, no los artefactos intermedios del proceso.

---

## Contribuciones

Este es un proyecto personal en evolución. Si encontrás algo que no funciona, querés proponer una mejora o adaptar el workflow a otro entorno, abrí un issue o un PR.

---

## Licencia

MIT
