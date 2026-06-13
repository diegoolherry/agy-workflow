---
name: chore
description: "Ejecuta tareas de mantenimiento (refactors, actualizaciones de dependencias, limpieza) sin requerir especificaciones Gherkin formales. Modifica el código, ejecuta tests, pide aprobación y commitea."
---

# Chore Skill

Esta skill permite al `chore_manager` ejecutar tareas técnicas directas sobre el código base, garantizando la estabilidad mediante la suite de tests y una pausa de revisión manual.

## Entradas Esperadas
1. El requerimiento del `chore` especificado por el leader o el usuario (ej: "actualizá la librería X a la versión Y", "formateá todos los archivos en la carpeta Z").

## Flujo de Ejecución

### 1. Ejecución del Código
1. Identifica y modifica los archivos necesarios para cumplir con el chore.
2. No es necesario crear archivos de `design.md` ni `requirements.md`. Modifica el código directamente.

### 2. Verificación de Regresiones
1. Ejecuta la suite de pruebas existente del proyecto en la terminal.
2. Si algún test falla, corrige tu implementación hasta que todos los tests pasen en verde. Si no logras que pasen después de 3 intentos, detente y reporta el bloqueo.

### 3. PAUSA DE REVISIÓN OBLIGATORIA
Una vez que el código esté listo y los tests en verde:
1. Comunica en el chat (usando `/caveman full`) que los cambios están listos para revisión y solicita aprobación:
   `"Código listo y tests en verde. Por favor revisa el git diff. ¿Aprobado?"`
2. **DETENTE.** No continúes hasta que el usuario responda explícitamente con su aprobación (ej: "aprobado", "ok", "dale"). Si el usuario pide cambios, vuelve al Paso 1 y repite el proceso.

### 4. Commit Atómico
1. Tras recibir la aprobación del usuario, ejecuta `git add .` (asegurándote de trackear solo los archivos correspondientes a este chore).
2. Haz un commit con el prefijo chore: `git commit -m "chore: [breve descripción del cambio]"`

### 5. Cierre
Reporta que la tarea finalizó al leader:
`done → chore commiteado exitosamente`
