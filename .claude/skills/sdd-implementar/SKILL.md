---
name: sdd-implementar
description: Implementa una spec aprobada con TDD estricto, criterio por criterio, respetando los archivos permitidos y los estándares de desarrollo. Usar después de que la spec está en estado Aprobada.
---

# Implementar: de la spec al código

Objetivo: que cada criterio de aceptación quede cubierto por al menos un test y que el código lo cumpla, sin salirse de la spec.

Este paso corre en un contexto aislado: **no hay historial de conversación**. Todo lo necesario está en los archivos que se leen a continuación.

## Reglas

- La spec es la única fuente de verdad. Lo que no está en la spec no se implementa.
- Solo se crean o modifican los archivos de "Archivos permitidos". Los únicos archivos extra que se pueden tocar son la propia spec (su estado y su commit base) y su fila en `docs/specs/STATUS.md`.
- No se hacen commits: la persona decide cuándo commitear después de verificar.
- Nunca se borra, se saltea ni se debilita un test para que pase.

## Paso 1 — Leer

1. La spec indicada en `docs/specs/`. Si su estado no es **Aprobada**, detenerse y reportarlo.
2. `AGENTS.md` y `docs/estandares/desarrollo.md`.
3. El mapa de impacto que la spec enlaza.
4. Los archivos permitidos que ya existen, y los tests relacionados.

## Paso 2 — Preparar

1. Identificar el comando de tests del proyecto a partir de su configuración (`pyproject.toml`, `package.json`, `Makefile` u otra), y la forma de correr un subconjunto (por archivo, por nombre o por marcador). Si no se puede determinar, detenerse y reportarlo.
2. **Durante el desarrollo se corren solo los tests de la spec**, no la suite completa. La suite completa se corre una sola vez, antes de cerrar (paso 4).
3. **Registrar el commit base:** obtener el commit actual (`git rev-parse HEAD`) y completarlo en el campo "Commit base" de la spec. `/verificar` compara los cambios contra ese commit.

## Paso 3 — Ciclo TDD por criterio

Recorrer los criterios en orden. Para cada uno:

1. **Rojo:** escribir el test que expresa el criterio. El nombre del test indica el escenario y el resultado esperado, y referencia el ID del criterio (por ejemplo, en el nombre o en un comentario `CA-3`).
2. **Correrlo y confirmar que falla por el motivo esperado** (la funcionalidad no existe o no se comporta así), no por un error de sintaxis o de configuración del test.
3. **Verde:** escribir la implementación **mínima** que hace pasar el test.
4. **Correr los tests de la spec** (los de este criterio y los de los criterios anteriores). Si alguno de los anteriores se rompió, arreglarlo antes de seguir.
5. **Refactor:** aplicar los estándares de desarrollo con los tests de la spec en verde, y volver a correrlos.

Los criterios de **regresión** suelen pasar desde el primer intento, porque describen comportamiento que ya existe. En ese caso, el test se escribe igual y se deja constancia de que pasó sin cambios de código.

## Paso 4 — Suite completa antes de cerrar

Cuando todos los criterios están implementados, correr **una vez** la suite completa para detectar regresiones fuera de la spec.

- Si hay fallas en tests que no son de la spec, comprobar si ya fallaban sin los cambios (por ejemplo, ejecutándolos sobre una copia limpia del commit actual).
  - **Ya fallaban:** informarlas como preexistentes y no arreglarlas.
  - **Las provocan los cambios:** son una regresión. Corregirla si cae dentro de los archivos permitidos; si no, detenerse y reportarlo.
- Repetir hasta que la única falla posible sea una preexistente.

## Cuándo detenerse

Detenerse, no improvisar y reportar el motivo si:

- hace falta modificar un archivo que no está en "Archivos permitidos";
- la spec es ambigua o contradictoria para algún criterio;
- un criterio no se puede cumplir sin romper otro;
- implementar exige una decisión de diseño con impacto fuera de la spec (nueva dependencia, cambio de esquema no previsto, cambio de contrato público).

## Autocontrol antes de terminar

- [ ] Cada criterio tiene al menos un test que lo referencia.
- [ ] Cada test se vio fallar antes de implementar, salvo los de regresión que ya pasaban.
- [ ] La suite completa se corrió una vez al final y está en verde, o las fallas restantes están comprobadas como preexistentes.
- [ ] Solo se tocaron archivos permitidos.
- [ ] El código cumple `docs/estandares/desarrollo.md`: sin abstracciones ni patrones que la spec no justifique.

## Cierre

1. Cambiar el estado de la spec a **Implementada** y actualizar su fila en `docs/specs/STATUS.md`.
2. Responder con un reporte breve:
   - tabla `criterio → test(s) → resultado`;
   - archivos creados y modificados;
   - resultado de la suite completa final, con las fallas preexistentes identificadas;
   - desvíos, dudas o deuda técnica detectada;
   - mensajes de commit sugeridos (conventional commits, en español).
3. **Detenerse**: la persona revisa el reporte y, al aprobarlo, el agente principal continúa con `sdd-verificar` sin pedir otra confirmación (ver `AGENTS.md`).
