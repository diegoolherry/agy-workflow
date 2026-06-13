---
name: rollback
description: "Deshace los cambios de una feature completada. Revierte los commits asociados, marca sus ADRs como revocados y genera el estado de reversión."
---

# Rollback Skill

Esta skill se usa exclusivamente para deshacer de forma limpia una feature completa que ya fue mergeada o implementada.

## Entrada Esperada
- ID o nombre de la feature a revertir (extraído de `feature_list.json` por el leader).

## Flujo de Reversión

### 1. Identificar Commits
1. Revisa el historial de Git para encontrar los commits asociados a la feature. Los commits de implementación siguen el formato `feat: [descripción] [T{n}]`. También puedes consultar `.agents/docs/logs/` para ver si hay un log de la feature que liste los commits exactos.
2. Anota los hashes de los commits que pertenecen EXCLUSIVAMENTE a esa feature. Si no encuentras commits distinguibles, reporta el error y detente.

### 2. Ejecutar Git Revert
1. Ejecuta `git revert --no-commit <hash1> <hash2> ...` (revirtiendo del más nuevo al más viejo).
2. Si hay conflictos, repórtalo en tu output para que el usuario o el leader intervengan. No intentes resolver conflictos complejos.
3. Haz un commit de la reversión: `git commit -m "revert: rollback de feature [nombre]"`

### 3. Revocar ADRs
1. Busca en `.agents/docs/architecture/decisions/` si existe algún ADR que haya sido creado explícitamente para esta feature (puedes guiarte por las fechas o menciones en el ADR).
2. Si encuentras un ADR asociado, edita el archivo y modifica o agrega el estado: `**Estado:** Revocado por rollback de feature [nombre]`.

### 4. Salida
Genera un archivo de resumen temporal: `.agents/docs/progress/rollback_<feature>.md` detallando:
- Commits revertidos.
- ADRs revocados (si hubo).
- Estado: Exitoso.

Reporta tu resultado al leader en el chat (recordando usar `/caveman full`) de esta forma:
`done → .agents/docs/progress/rollback_<feature>.md`
