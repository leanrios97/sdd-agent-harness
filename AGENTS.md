# Contrato de trabajo

Reglas que todo agente sigue en este repositorio, sin importar la herramienta (Claude Code u OpenCode).

## Flujo

Todo cambio sigue tres pasos. Al terminar cada uno, el agente se detiene y espera aprobación explícita antes de continuar.

1. **`/spec`** — Escribir la spec del cambio en `docs/specs/` a partir de `docs/templates/spec.md`.
2. **`/implementar`** — El agente desarrollador implementa la spec aprobada con TDD estricto.
3. **`/verificar`** — Comprobar cada criterio de aceptación con evidencia.

## Reglas

### Specs
- No se escribe código de producción sin una spec aprobada.
- La spec es la única fuente de verdad del cambio. Si algo no está en la spec, no se implementa.
- Si durante el trabajo aparece algo que la spec no contempla, el agente se detiene y propone actualizar la spec.

### Alcance
- Solo se crean o modifican los archivos listados en la sección "Archivos permitidos" de la spec.
- Si hace falta tocar otro archivo, el agente se detiene, explica por qué y espera aprobación.

### TDD
- Primero un test que falle y que exprese un criterio de aceptación.
- Después la implementación mínima que lo haga pasar.
- Por último, refactor con todos los tests en verde.
- Cada criterio de aceptación tiene al menos un test.

### Calidad
- El código sigue los estándares de `docs/estandares/desarrollo.md`.
- Ante dos soluciones que cumplen la spec, se elige la más simple.

### Commits
- Conventional commits, con la descripción en español.
- Sin atribución de IA ni líneas `Co-Authored-By`.
- Un commit por unidad de trabajo coherente; los tests van junto con el código que prueban.

## Idioma

- Documentación, specs, comentarios y mensajes de commit: español neutro.
- Identificadores de código (variables, funciones, clases, archivos de código): inglés.

## Comunicación

- Respuestas breves y concretas.
- Ante una duda que cambia el resultado, preguntar antes de asumir.
- Informar los resultados tal como son: si un test falla o un paso se omitió, decirlo.
