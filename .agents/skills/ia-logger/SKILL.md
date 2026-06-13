---

name: ia-logger
description: "Ejecutar SIEMPRE como último paso al terminar cualquier tarea de desarrollo: features, bugfixes, refactors, migraciones, o cualquier cambio concreto al código. El agente lo ejecuta automáticamente sin esperar que el usuario lo pida. No registrar tareas triviales de un solo archivo y menos de 5 líneas de código (ej: corregir un typo, renombrar una variable)."

---

# ia-logger — Registro automático de tareas

Al finalizar cada tarea, crear un archivo de log en `.agents/docs/logs/`.

## Comportamiento

Este skill se ejecuta **automáticamente al finalizar cada tarea**, como último paso. No requiere instrucción del usuario.

**Objetivo:** mantener `.agents/docs/logs/` actualizado con un registro de cada tarea completada, con suficiente contexto para entender qué se hizo, por qué, y cómo contribuyó la IA.

---

## Paso 1 — Determinar el número de archivo

Listar los archivos existentes en `.agents/docs/logs/`:

```bash
ls .agents/docs/logs/
```

- Si la carpeta no existe → crearla:
  ```bash
  mkdir -p .agents/docs/logs
  ```
- Tomar el número más alto existente e incrementar en 1.
- Si no hay archivos → empezar desde `01`.

---

## Paso 2 — Crear el archivo

**Nombre:** `{numero_dos_digitos}-{descripcion-corta}.md`  
Ejemplos: `01-scaffold-inicial.md`, `02-entidad-usuario.md`, `03-fix-validacion-login.md`

La descripción corta debe ser kebab-case, máximo 4 palabras, descriptiva de la tarea.

**Ruta completa:** `.agents/docs/logs/{nombre-archivo}.md`

---

## Formato del archivo (máximo 60 líneas)

```markdown
# {número} — {Descripción breve de la tarea}

**Fecha:** DD/MM/YYYY  
**Hora:** HH:MM  
**Modelo:** {modelo activo en esta sesión}

## Tipo de tarea
[feature / bugfix / refactor / otro]

## Qué se hizo
{3-6 líneas. Qué archivos se crearon o modificaron y para qué.}

## Decisión clave
{2-4 líneas. Solo si hubo una decisión de arquitectura, diseño o técnica relevante.
Omitir esta sección si no aplica.}

## Problemas encontrados
{2-4 líneas. Errores, bugs o bloqueos que surgieron durante la tarea y cómo se resolvieron.
Omitir esta sección si no hubo ninguno.}

## Cómo contribuyó la IA
{2-4 líneas. Qué generó el agente, qué ajustó o revisó el usuario manualmente.}

## Estado
[Completado / Completado con pendientes / Bloqueado]

## Métricas
- **Code Coverage:** {Extraer de impl_<feature>.md si existe}
- **Uso de /diagnose:** {Extraer de impl_<feature>.md si existe}
- **Intentos de review:** {Extraer de review_<feature>.md si existe}

## Pendientes
{Lista de lo que quedó sin resolver. Omitir esta sección si no hay pendientes.}
```

---

## Reglas

- El número es **secuencial y nunca se reutiliza**. Siempre verificar los existentes antes de asignar uno nuevo.
- **Máximo 60 líneas** por archivo. Si hay mucho que registrar, priorizar lo más relevante.
- Las secciones marcadas como "Omitir si no aplica" deben eliminarse del archivo cuando no correspondan — no dejar encabezados vacíos.
- No registrar tareas triviales: un solo archivo modificado, menos de 5 líneas de código, sin decisión relevante.
- La hora debe ser la hora local al momento de finalizar la tarea.
- El modelo debe ser el que estuvo activo durante la sesión, no inferirlo — si no se sabe con certeza, escribir el que corresponda al agente que ejecuta el skill.

---

## Ejemplo

```markdown
# 03 — Validación de login con JWT

**Fecha:** 03/06/2026  
**Hora:** 14:35  
**Modelo:** Claude Sonnet 4.6

## Tipo de tarea
Feature

## Qué se hizo
Se creó `AuthService.cs` con el método `ValidarTokenAsync` que verifica firma,
expiración y claims del JWT. Se modificó `LoginController.cs` para delegar
la validación al servicio. Se agregó el middleware de autenticación en `Program.cs`.

## Decisión clave
Se optó por validación stateless sin blacklist de tokens. La expiración corta
(15 min) con refresh token cubre el caso de logout sin necesidad de estado servidor.

## Problemas encontrados
El middleware no interceptaba rutas con `[AllowAnonymous]` correctamente.
Se resolvió reordenando el pipeline en `Program.cs` — `UseAuthentication` debe
ir antes de `UseAuthorization`.

## Cómo contribuyó la IA
El agente generó `AuthService.cs` completo y el middleware. La corrección del
orden en el pipeline fue identificada por el agente al analizar el error de runtime.

## Estado
Completado

## Métricas
- **Code Coverage:** 92%
- **Uso de /diagnose:** No
- **Intentos de review:** 2
```
