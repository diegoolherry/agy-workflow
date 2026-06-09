---
name: spec-design
description: "Escribe la especificación de diseño técnico de una feature (design.md). Úsala como el Paso 2 en la creación de specs."
---

# Spec Design

Eres responsable de redactar las decisiones técnicas y el diseño de la solución, basándote en los requerimientos.

## Reglas Principales

- **Solo Escritura de Design:** Generarás únicamente el archivo `design.md`.
- **Coherencia Arquitectónica:** Las decisiones deben respetar `architecture.md`, el research previo, y los ADRs en `.agents/docs/architecture/decisions/`. Si una decisión de diseño contradice un ADR existente, debe documentarlo explicitamente y justificar la excepción.

## Entradas Esperadas

Antes de ejecutar esta skill, debes haber leído o generado:
1. `.agents/docs/specs/{feature}/requirements.md` (Paso 1).
2. `.agents/docs/progress/researchs/research_{feature}.md`.

## Salida Esperada

Escribe el archivo en `.agents/docs/specs/{feature}/design.md` siguiendo ESTRICTAMENTE este formato:

```markdown
# Design: {feature}

## Decisiones técnicas
*Lista con justificación breve para cada una. Relaciona estas decisiones con el research previo.*

## Cambios al modelo de datos
*Tablas, entidades o relaciones nuevas o modificadas. Omitir si no aplica.*

## Interfaces y contratos
*Endpoints, métodos públicos o eventos con firmas concretas.*

## Dependencias
*Librerías, servicios externos u otros módulos involucrados.*
```

Al terminar, reporta `done → design.md` y prepárate para el siguiente paso del proceso.
