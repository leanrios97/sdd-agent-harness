---
name: sdd-verificar
description: Verifica con evidencia que una spec implementada cumple cada criterio de aceptación, los archivos permitidos y los estándares de desarrollo. Usar después de /implementar y antes de commitear.
---

# Verificar: evidencia, no confianza

Objetivo: comprobar de forma independiente que lo implementado cumple la spec. El reporte del desarrollador es una pista, no una prueba: todo se vuelve a comprobar.

## Reglas

- Solo lectura sobre el código y los tests. No se corrige nada en este paso: se reporta.
- Las únicas escrituras permitidas son el reporte de verificación, el cambio de estado de la spec y su fila en `docs/specs/STATUS.md`.
- Cada veredicto lleva evidencia concreta (`ruta:línea`, salida de un comando). Sin evidencia, el veredicto es "no verificado".
- No se asume que un test prueba lo que dice su nombre: se lee.

## Paso 1 — Leer

1. La spec indicada en `docs/specs/`. Si su estado no es **Implementada**, detenerse y reportarlo.
2. `docs/estandares/desarrollo.md`.
3. El mapa de impacto que la spec enlaza.
4. El diff de los cambios contra el **commit base** de la spec: `git diff <commit base>` más `git status` para los archivos sin seguimiento. Si la spec no registra commit base, usar `git status` y `git diff`.

## Paso 2 — Alcance

Comparar los archivos modificados con "Archivos permitidos". Quedan fuera de este control los archivos del propio flujo: la spec, su mapa de impacto, su reporte de verificación y `docs/specs/STATUS.md`.

- Archivo modificado que no está permitido → **hallazgo bloqueante**.
- Archivo permitido que no se tocó → anotarlo y verificar que ningún criterio dependía de él.

## Paso 3 — Ejecutar la suite

1. Correr la suite completa y registrar el comando y el resultado.
2. Comparar con las fallas preexistentes que informó la implementación. Toda falla que no esté en esa lista es un hallazgo bloqueante.

## Paso 4 — Verificar cada criterio

Para cada criterio de la spec:

1. Localizar el o los tests que lo cubren. Si no hay ninguno, el criterio es **no cumple**.
2. **Leer el test** y comprobar que:
   - la preparación corresponde al "Dado";
   - la acción corresponde al "Cuando";
   - las aserciones corresponden al "Entonces" y no son más débiles (por ejemplo, comprobar solo que no lanza error cuando el criterio exige un valor concreto).
3. Confirmar que el test pasa en la corrida del paso 3.
4. Leer el código implementado y comprobar que el comportamiento está realmente ahí y no hay un caso especial hecho a medida para que el test pase.
5. Asignar el veredicto:
   - **Cumple:** test adecuado, en verde, con el código que lo respalda.
   - **Cumple parcialmente:** el test es más débil que el criterio o cubre solo una parte.
   - **No cumple:** falta el test, falla o el comportamiento es distinto.
   - **No verificado:** no se pudo comprobar, con el motivo.

## Paso 5 — Estándares

Revisar el código nuevo y modificado contra `docs/estandares/desarrollo.md`. Registrar cada incumplimiento con su ubicación y clasificarlo:

- **Bloqueante:** rompe una regla de arquitectura (el dominio depende de infraestructura), duplica lógica de negocio, silencia errores, o agrega una abstracción o patrón que la spec no justifica.
- **Menor:** nombres poco claros, funciones largas, comentarios que explican el qué.

## Paso 6 — Tests y deuda

- Tests no deterministas (hora, orden, red) o que dependen de detalles de implementación.
- Tests deshabilitados, saltados o debilitados en la diff.
- Deuda técnica o dudas que el desarrollador dejó en su reporte.

## Resultado

Escribir `docs/verificaciones/<ID>-<slug>.md` con:

1. **Veredicto global:**
   - **Aprobada:** todos los criterios cumplen y no hay hallazgos bloqueantes.
   - **Con observaciones:** todos cumplen y solo hay hallazgos menores.
   - **Rechazada:** al menos un criterio no cumple o hay un hallazgo bloqueante.
2. Tabla `criterio → test(s) → veredicto → evidencia`.
3. Resultado de la suite y comando usado.
4. Resultado del control de alcance.
5. Hallazgos, ordenados por gravedad y con ubicación.
6. Mensajes de commit sugeridos, solo si el veredicto es **Aprobada** o **Con observaciones**.

## Cierre

- Si el veredicto es **Aprobada** o **Con observaciones**, cambiar el estado de la spec a **Verificada** y actualizar su fila en `docs/specs/STATUS.md`, enlazando el reporte de verificación.
- Si es **Rechazada**, dejar el estado en **Implementada** y listar qué debe corregir `/implementar`.
- Responder con un resumen breve y la ruta del reporte, y **detenerse**. El commit lo decide la persona.
