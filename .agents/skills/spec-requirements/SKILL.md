---
name: spec-requirements
description: "Escribe la especificación de requerimientos de una feature (requirements.md). Úsala como el Paso 1 en la creación de specs."
---

# Spec Requirements

Eres responsable de redactar los requerimientos funcionales y criterios de aceptación.

## Reglas Principales

- **Solo Escritura de Requirements:** Generarás únicamente el archivo `requirements.md`. No escribas diseño ni tasks aquí.
- **Formato Gherkin:** Los criterios de aceptación deben usar Gherkin estricto.

## Entradas Esperadas

Antes de ejecutar esta skill, debes haber leído:
1. `.agents/docs/progress/researchs/research_{feature}.md`
2. `.agents/docs/architecture.md` (y los dominios relevantes).
3. `.agents/docs/architecture/decisions/` — los requerimientos no pueden contradecir restricciones ya documentadas como ADR.

## Salida Esperada

Escribe el archivo en `.agents/docs/specs/{feature}/requirements.md` siguiendo ESTRICTAMENTE este formato:

```markdown
# Requirements: {feature}

## Contexto
*1 a 2 líneas explicando qué problema resuelve esta feature, extraído del research.*

## Criterios de aceptación
*Formato Gherkin, uno por escenario.*

  Scenario: [nombre del escenario]
    Given [estado inicial del sistema]
    When  [acción del usuario o del sistema]
    Then  [resultado esperado verificable]

*Debes cubrir obligatoriamente: camino feliz, edge cases principales y casos de error.*

## No-comportamientos
*Qué NO debe hacer esta feature, explícito y sin ambigüedad.*
```

Al terminar, reporta `done → requirements.md` y prepárate para el siguiente paso del proceso.
