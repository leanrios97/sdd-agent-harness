# Contrato de trabajo

Reglas que todo agente sigue en este repositorio, sin importar la herramienta (Claude Code u OpenCode).

## Flujo

Todo cambio sigue cuatro pasos. Al terminar cada uno, el agente presenta el resultado y espera la aprobación de la persona.

1. **`/explorar`** — El agente explorador construye el mapa de impacto en `docs/exploraciones/` a partir de `docs/templates/exploracion.md`.
2. **`/spec`** — Escribir la spec del cambio en `docs/specs/` a partir de `docs/templates/spec.md` y del mapa de impacto.
3. **`/implementar`** — El agente desarrollador implementa la spec aprobada con TDD estricto.
4. **`/verificar`** — Comprobar cada criterio de aceptación con evidencia.

### Aprobación y continuidad

La aprobación de un paso inicia el siguiente de inmediato: no hay que escribir el comando ni volver a confirmar. Una sola respuesta de la persona ("aprobado", "ok", "dale") alcanza.

| La persona aprueba | El agente principal continúa con |
|--------------------|----------------------------------|
| El mapa de impacto | `sdd-spec` |
| La spec | Delegar en el agente `desarrollador` para ejecutar `sdd-implementar` |
| La implementación | `sdd-verificar` |

- Si la persona pide cambios en lugar de aprobar, se corrige el resultado y se vuelve a pedir aprobación.
- Tras `/verificar` el flujo termina: el commit lo decide la persona.

### Ciclo de vida de las specs

| Estado | Se llega cuando | Lo cambia |
|--------|-----------------|-----------|
| Borrador | Se escribe la spec | `/spec` |
| Aprobada | La persona aprueba | Agente principal |
| Implementada | La implementación termina | `/implementar` |
| Verificada | La verificación da "Aprobada" o "Con observaciones" | `/verificar` |
| Bloqueada | Falta una decisión para avanzar | Agente principal |
| Descartada | La persona decide no continuar (solo desde Borrador o Aprobada) | Agente principal |

- Una verificación "Rechazada" deja la spec en Implementada hasta que se corrija.
- Una spec Bloqueada vuelve al estado anterior cuando la decisión se resuelve.
- Si una spec Aprobada o más avanzada cambia, vuelve a Borrador y requiere una nueva aprobación.
- No se permiten transiciones fuera de esta tabla.

### Índice de specs

`docs/specs/STATUS.md` lista todas las specs con su estado. Cada cambio de estado actualiza la fila correspondiente en el mismo paso, y `/estado` lo usa para informar dónde quedó el trabajo. Se crea a partir de `docs/templates/status.md` cuando se escribe la primera spec.

## Reglas

### Exploración
- No se escribe una spec sin un mapa de impacto aprobado.
- Cada categoría del mapa se completa siempre; si no hay hallazgos, se indica "revisado: ninguno".

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
