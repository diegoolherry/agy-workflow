---
name: tdd-implement
description: "Implementa features siguiendo TDD estricto (RED → GREEN → REFACTOR). Maneja la ejecución real de tests en terminal y genera commits atómicos por task. Úsala siempre que actúes como Implementer."
---

# TDD Implement

Eres responsable de implementar una feature ejecutando una por una las tareas del archivo `tasks.md`, validando el comportamiento con tests reales y guardando el progreso en Git.

## Entradas Esperadas

Antes de empezar, debes haber leído:
1. `.agents/docs/specs/{feature}/requirements.md`
2. `.agents/docs/specs/{feature}/design.md`
3. `.agents/docs/specs/{feature}/tasks.md`
4. `.agents/docs/architecture.md` (y dominios relevantes si existen)
5. `.agents/docs/architecture/decisions/` — si hay ADRs relacionados al área que se va a modificar, son restricciones que no se pueden violar.

## Paso 0: Verificación de Entorno (Pre-flight)

1. **Árbol de trabajo limpio:** Ejecuta `git status`. Si hay archivos modificados sin commitear, **DETENTE** y avisa al líder/usuario: *"Hay cambios sin commitear en el repositorio. Por favor límpialos o haz commit antes de que comience a implementar."* No avances hasta que el working tree esté limpio.
2. **Detección del comando de test:** Inspecciona el proyecto (ej. `package.json`, `pytest.ini`, `Makefile`) para determinar cómo ejecutar los tests. Si no estás seguro, consúltalo antes de empezar.

## Flujo TDD por Task

Por cada task en `tasks.md`, repite este ciclo ESTRICTAMENTE en orden:

### 1. RED (Fase de Fallo)
- Escribe un test unitario o de integración que defina el comportamiento de la task.
- **Ejecuta el test en la terminal** usando el comando detectado.
- Verifica en la salida de la consola que el test **FALLA** por la razón correcta (aún no está implementado).

### 2. GREEN (Fase de Implementación)
- Escribe el código mínimo e indispensable de producción para que el test pase. Nada más. No te salgas del scope definido en `design.md`.
- **Ejecuta el test en la terminal.**
- Verifica en la salida de la consola que el test **PASA**.

### 3. REFACTOR (Fase de Limpieza - Opcional pero Restringido)
- Si el código que acabas de escribir en la fase GREEN se puede mejorar en legibilidad o estructura, hazlo ahora.
- **RESTRICCIÓN:** Solo puedes limpiar o mejorar la función/archivo que acabas de modificar. No tienes permitido extraer lógica a nuevos archivos o modificar otras dependencias para reducir duplicación global.
- **Ejecuta el test en la terminal** para verificar que sigue en verde tras el refactor.

### 4. CHECKPOINT (Commit Atómico)
- Ejecuta `git add .` para trackear los cambios de la task actual.
- Ejecuta `git commit -m "feat: [descripción de la task] [T{n}]"`. Reemplaza `{n}` con el número de task.
- Marca la task como completada en `tasks.md`: `- [x] T{n}: ...`

## Reglas de Bloqueo

- Si una task falla (no puedes lograr el GREEN) después de 3 intentos de corrección, invoca la skill `/diagnose` para analizar el bloqueo.
- Si tras usar `/diagnose` sigues sin resolverlo, escribe en tu reporte: `BLOQUEADO en T{n}: [descripción]` y detente.

## Salida Esperada

Mantén actualizado el archivo `.agents/docs/progress/impl/impl_{feature}.md` con el siguiente formato:

```markdown
# Implementación: {feature}

## Tasks completadas
*Lista con referencia a archivos modificados y el hash/mensaje del commit asociado.*

## Tests escritos
*Lista con archivo:línea de cada test.*

## Decisiones tomadas durante la implementación
*Desviaciones de la spec o decisiones no previstas, con justificación.*

## Métricas de Ejecución
- **Code Coverage final:** *[Ejecuta toda la suite de tests con --coverage al finalizar y anota el %]*
- **Uso de /diagnose:** *[Sí (N veces) / No]*

## Estado
COMPLETO | BLOQUEADO en T{n}: [descripción]
```

Al finalizar todas las tasks (o al bloquearte permanentemente), reporta únicamente:
`done → .agents/docs/progress/impl/impl_{feature}.md`
