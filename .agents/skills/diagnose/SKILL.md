---
name: diagnose
description: "Disciplined debugging loop for hard bugs and performance regressions. Reproduce → minimise → hypothesise → instrument → fix → regression-test. Use when user says 'diagnose this' / 'debug this' / 'investigate this', reports a bug, says something is broken/throwing/failing, describes unexpected behavior, or describes a performance regression. Invocar proactivamente cuando el usuario reporta errores, stack traces, comportamiento inesperado, o 'funcionaba ayer'."
---

# Diagnose

Una disciplina para bugs difíciles. Saltear fases solo cuando esté explícitamente justificado.

Al explorar el codebase, usar el glosario de dominio del proyecto para tener un modelo mental claro de los módulos relevantes, y revisar `docs/architecture.md` si existe. Si existen ADRs en `.agents/docs/architecture/decisions/`, revisarlos — a veces una decisión arquitectónica "rara" no es un bug sino una restricción intencional documentada.

---

## Fase 1 — Construir un feedback loop

**Esta es la habilidad central.** Todo lo demás es mecánico. Si tenés una señal pass/fail rápida, determinista y ejecutable por el agente para el bug, vas a encontrar la causa — bisección, testing de hipótesis e instrumentación solo consumen esa señal. Si no la tenés, ninguna cantidad de mirar código te va a salvar.

Dedicar esfuerzo desproporcionado aquí. **Ser agresivo. Ser creativo. No rendirse.**

### Formas de construirlo — intentar en este orden

1. **Failing test** en cualquier seam que llegue al bug — unit, integration, e2e.
2. **Curl / HTTP script** contra un dev server corriendo.
3. **CLI invocation** con un input fixture, diffing stdout contra un snapshot conocido.
4. **Replay de un trace capturado.** Guardar un request real / payload / event log a disco; reproducirlo a través del code path en aislamiento.
5. **Throwaway harness.** Levantar un subconjunto mínimo del sistema que ejercite el code path del bug con una sola llamada de función.
6. **Property / fuzz loop.** Si el bug es "output a veces incorrecto", correr 1000 inputs random y buscar el failure mode.
7. **Bisection harness.** Si el bug apareció entre dos estados conocidos (commit, dataset, versión), automatizar "boot en estado X, check, repetir" para poder usar `git bisect run`.
8. **Differential loop.** Correr el mismo input en versión vieja vs nueva (o dos configs) y diffear outputs.

Construir el feedback loop correcto, y el bug está 90% resuelto.

### Iterar sobre el loop

Tratar el loop como un producto. Una vez que tenés uno, preguntar:

- ¿Puedo hacerlo más rápido? (Cache del setup, saltear init no relacionado, acotar el scope del test.)
- ¿Puedo hacer la señal más nítida? (Assert sobre el síntoma específico, no "no crasheó".)
- ¿Puedo hacerlo más determinista? (Pin de tiempo, seed de RNG, aislar filesystem, freeze de red.)

Un loop de 30 segundos inestable es apenas mejor que no tener loop. Un loop de 2 segundos determinista es un superpoder de debugging.

### Bugs no deterministas

El objetivo no es una reproducción limpia sino una **tasa de reproducción más alta**. Loopearlo 100×, paralelizar, agregar stress, acotar timing windows, inyectar sleeps. Un bug que falla 50% del tiempo es debuggeable; 1% no lo es — seguir subiendo la tasa hasta que sea debuggeable.

### Cuando genuinamente no se puede construir un loop

Parar y decirlo explícitamente. Listar qué se intentó. Pedir al usuario: (a) acceso al entorno que lo reproduce, (b) un artefacto capturado (HAR file, log dump, core dump, screen recording con timestamps), o (c) permiso para agregar instrumentación temporal de producción. **No** proceder a hipotetizar sin un loop.

No avanzar a la Fase 2 sin tener un loop en el que se confíe.

---

## Fase 2 — Reproducir

Correr el loop. Ver aparecer el bug.

Confirmar:

- [ ] El loop produce el failure mode que **el usuario** describió — no un failure diferente que esté cerca. Bug incorrecto = fix incorrecto.
- [ ] El failure es reproducible en múltiples runs (o, para bugs no deterministas, reproducible a una tasa suficientemente alta para debuggear).
- [ ] Se capturó el síntoma exacto (mensaje de error, output incorrecto, timing lento) para que las fases siguientes puedan verificar que el fix realmente lo resuelve.

No avanzar hasta reproducir el bug.

---

## Fase 3 — Hipotetizar

Antes de testear cualquier hipótesis, chequear si el bug matchea un patrón conocido:

| Patrón | Firma | Dónde mirar |
|--------|-------|-------------|
| Race condition | Intermitente, dependiente del timing | Acceso concurrente a estado compartido |
| Null propagation | NullReferenceException, TypeError | Guards faltantes en valores opcionales |
| State corruption | Datos inconsistentes, actualizaciones parciales | Transacciones, callbacks, hooks |
| Integration failure | Timeout, respuesta inesperada | Llamadas a APIs externas, service boundaries |
| Configuration drift | Funciona local, falla en staging/prod | Variables de entorno, feature flags, estado de DB |
| Stale cache | Muestra datos viejos, se arregla limpiando cache | Redis, CDN, browser cache |

Si el bug matchea un patrón → usarlo como hipótesis candidata principal.

Luego generar **3–5 hipótesis rankeadas**. La generación de hipótesis única ancla en la primera idea plausible.

Cada hipótesis debe ser **falsificable**: declarar la predicción que hace.

> Formato: "Si <X> es la causa, entonces <cambiar Y> hará que el bug desaparezca / <cambiar Z> lo empeorará."

Si no se puede declarar la predicción, la hipótesis es un vibe — descartarla o afilarla.

**Mostrar la lista rankeada al usuario antes de testear.** Frecuentemente tienen conocimiento de dominio que re-rankea instantáneamente. Checkpoint barato, gran ahorro de tiempo. No bloquear en ello — proceder con el ranking si el usuario no responde.

---

## Fase 4 — Instrumentar

Cada probe debe mapear a una predicción específica de la Fase 3. **Cambiar una variable a la vez.**

Preferencia de herramienta:

1. **Debugger / REPL inspection** si el entorno lo soporta. Un breakpoint vale más que diez logs.
2. **Logs dirigidos** en los boundaries que distinguen hipótesis.
3. Nunca "loggear todo y grep".

**Taggear cada debug log** con un prefijo único, ej: `[DEBUG-a4f2]`. La limpieza al final se convierte en un solo grep. Los logs sin tag sobreviven; los taggeados mueren.

**Rama de perf.** Para regresiones de performance, los logs suelen ser incorrectos. En cambio: establecer una medición baseline (timing harness, profiler, query plan), luego bisectar. Medir primero, fijar después.

### Regla de 3 strikes

Si 3 hipótesis fallan, **PARAR**. Preguntar al usuario:

```
3 hipótesis testeadas, ninguna matchea. Esto puede ser un problema arquitectural
más que un bug simple.

A) Continuar investigando — tengo una nueva hipótesis: [describir]
B) Escalar para revisión humana — esto necesita alguien que conozca el sistema
C) Agregar logging y esperar — instrumentar el área y capturarlo la próxima vez
```

---

## Fase 5 — Fix + regression test

Escribir el regression test **antes del fix** — pero solo si existe un **seam correcto** para él.

Un seam correcto es uno donde el test ejercita el **patrón real del bug** tal como ocurre en el call site.

**Fix the root cause, not the symptom.** El cambio más pequeño que elimina el problema real. Mínima diff: menos archivos tocados, menos líneas cambiadas. Resistir el impulso de refactorizar código adyacente.

Si existe un seam correcto:

1. Convertir el repro minimizado en un failing test en ese seam.
2. Verlo fallar.
3. Aplicar el fix.
4. Verlo pasar.
5. Re-correr el feedback loop de la Fase 1 contra el escenario original (no minimizado).

**Si el fix toca más de 5 archivos:** alertar al usuario antes de proceder:

```
Este fix toca N archivos. Eso es un blast radius grande para un bugfix.
A) Proceder — la causa raíz genuinamente abarca estos archivos
B) Dividir — fijar el critical path ahora, diferir el resto
C) Repensar — tal vez hay un enfoque más acotado
```

**Si no existe seam correcto:** notarlo. La arquitectura del codebase está impidiendo que el bug sea encerrado. Flaggearlo para post-mortem.

---

## Fase 6 — Cleanup + post-mortem

Requerido antes de declarar done:

- [ ] El repro original ya no se reproduce (re-correr el loop de la Fase 1)
- [ ] El regression test pasa (o la ausencia de seam está documentada)
- [ ] Toda la instrumentación `[DEBUG-...]` removida (`grep` del prefijo)
- [ ] Los throwaway prototypes borrados (o movidos a una ubicación claramente marcada)
- [ ] La hipótesis que resultó correcta está declarada en el commit / PR message

**Luego preguntar: ¿qué habría prevenido este bug?** Si la respuesta involucra cambio arquitectural (sin buen test seam, callers enredados, acoplamiento oculto), hacer la recomendación **después** del fix, no antes — se tiene más información ahora que cuando se empezó.

---

## Reporte final

Al cerrar la tarea, emitir siempre este reporte:

```
DEBUG REPORT
════════════════════════════════════════
Síntoma:          [lo que el usuario observó]
Causa raíz:       [lo que estaba realmente mal]
Fix:              [qué se cambió, con referencias archivo:línea]
Evidencia:        [output del test, intento de reproducción mostrando que el fix funciona]
Regression test:  [archivo:línea del nuevo test, o "sin seam correcto — documentado"]
Notas:            [items de deuda técnica, bugs previos en la misma área, notas arquitecturales]
Estado:           DONE | DONE_WITH_CONCERNS | BLOQUEADO
════════════════════════════════════════
```

Estados:
- **DONE** — causa raíz encontrada, fix aplicado, regression test escrito, todos los tests pasan.
- **DONE_WITH_CONCERNS** — fixeado pero no se puede verificar completamente (ej: bug intermitente, requiere staging).
- **BLOQUEADO** — causa raíz poco clara después de la investigación, escalado.

---

## Reglas

- Sin loop de reproducción, no se hipotetiza. Sin hipótesis confirmada, no se fixa.
- Nunca decir "esto debería arreglarlo". Verificar y probarlo.
- 3 hipótesis fallidas → parar y cuestionar la arquitectura, no generar una cuarta.
- Fix que toca más de 5 archivos → alertar al usuario antes de proceder.
- Nunca aplicar un fix que no se puede verificar.
- Los estados de completion son exactamente tres: DONE, DONE_WITH_CONCERNS, BLOQUEADO.
