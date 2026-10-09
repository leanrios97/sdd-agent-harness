# ADR-0001: No adoptar OpenSpec ni Spec Kit como base del harness

- **Estado:** Aceptada
- **Fecha:** 2026-10-08

## Contexto

El harness busca mejorar el ciclo de desarrollo con un flujo guiado por especificaciones (SDD) que sea **rápido, simple y propio**, y que funcione en Claude Code y OpenCode. Cada fase y cada artefacto del flujo tiene que justificar su costo en tiempo.

Existen dos frameworks de specs populares que podrían servir de base:

| | OpenSpec | Spec Kit |
|---|---|---|
| Autor | Fission AI | GitHub |
| Flujo | explore → propose → apply → archive | constitution → specify → plan → tasks → implement → converge |
| Artefactos por cambio | Propuesta, specs, diseño y lista de tareas | Un artefacto por fase, más una constitución del proyecto |
| Instalación | CLI sobre Node.js 20.19+ | CLI sobre Python 3.11+ y uv |
| Licencia | MIT | MIT |

## Decisión

No adoptar ninguno de los dos como dependencia ni como estructura base. El harness define su propio flujo y formato de specs, y toma de forma explícita conceptos puntuales de cada uno.

## Motivos

1. **Ceremonia.** Ambos imponen varias fases y artefactos por cambio. El objetivo de la v0.1 es lo contrario: un flujo de tres pasos (`/spec`, `/implementar`, `/verificar`) con un solo documento por cambio.
2. **Dependencias de runtime.** Ambos requieren un CLI (Node.js o Python). El harness es Markdown y configuración de cada herramienta, sin nada que instalar.
3. **Control del diseño.** Construir sobre un framework externo fija su modelo de trabajo y sus nombres. El harness necesita poder cambiar el flujo según lo que demuestre el uso real.
4. **Complejidad incremental.** Los frameworks llegan completos desde el día uno. El harness suma piezas por versión, solo cuando hacen falta.

## Conceptos tomados

| Concepto | Origen | Cómo se aplica en el harness |
|----------|--------|------------------------------|
| Constitución: principios que se respetan en todo cambio | Spec Kit | `AGENTS.md` + `docs/estandares/desarrollo.md` (v0.1) |
| Separar specs vigentes de cambios en curso, con archivo de los cambios terminados | OpenSpec | Se evaluará en v0.2, junto con el estado de las specs |
| Escenarios verificables por requisito (WHEN/THEN) | OpenSpec | Criterios de aceptación verificables en la plantilla de spec (v0.1) |

## Consecuencias

**Positivas**
- Flujo más corto y sin dependencias de instalación.
- Libertad para ajustar el diseño con cada versión.
- El harness refleja decisiones propias y documentadas.

**Negativas**
- Se pierde la madurez y el soporte de comunidad de proyectos con mucha adopción.
- Hay que mantener los adaptadores para Claude Code y OpenCode, que estos frameworks ya resuelven.
- Las mejoras futuras de esos frameworks no llegan solas; hay que revisarlas y decidir si se incorporan.

## Revisión

Revisar esta decisión si el formato propio de specs empieza a reimplementar lo que ya resuelve alguno de los frameworks, o si el costo de mantener los adaptadores supera el beneficio de un flujo propio.

## Referencias

- OpenSpec: https://github.com/Fission-AI/OpenSpec
- Spec Kit: https://github.com/github/spec-kit
