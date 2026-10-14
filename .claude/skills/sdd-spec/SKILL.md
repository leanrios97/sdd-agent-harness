---
name: sdd-spec
description: Escribe la spec de un cambio a partir de un mapa de impacto aprobado, con criterios de aceptación verificables en formato Dado/Cuando/Entonces. Usar después de /explorar y antes de /implementar.
---

# Spec: qué debe cumplir el cambio

Objetivo: convertir el pedido y el mapa de impacto en un contrato verificable. La spec dice **qué** tiene que pasar; el **cómo** lo decide el desarrollador.

## Reglas

- Partir siempre de un mapa de impacto aprobado en `docs/exploraciones/`. Si no existe, detenerse y pedir `/explorar`.
- No inventar requisitos. Si algo no está claro, preguntar.
- No incluir decisiones de implementación (clases, patrones, librerías) en los criterios.
- Cada criterio tiene que poder convertirse en un test.
- Cada cambio de estado actualiza la fila de la spec en `docs/specs/STATUS.md`.
- Si la spec ya estaba Aprobada o más avanzada y cambia, vuelve a **Borrador** y requiere una nueva aprobación.

## Paso 1 — Leer

- El mapa de impacto indicado.
- `docs/templates/spec.md`.
- El pedido original, tal como figura en el mapa.

## Paso 2 — Resolver ambigüedades

Listar todo lo que el pedido y el mapa no dejan claro y clasificarlo:

- **Bloqueante:** distintas respuestas producen comportamientos distintos. Ejemplo: "¿un usuario desactivado conserva sus sesiones abiertas?".
- **No bloqueante:** no cambia los criterios. Ejemplo: el texto exacto de un mensaje.

Si hay bloqueantes, **preguntarlas todas juntas, numeradas, antes de escribir la spec**, y esperar las respuestas. Las no bloqueantes van a "Notas".

## Paso 3 — Escribir la spec

1. Usar el mismo ID y slug que la exploración: `docs/specs/<ID>-<slug>.md`, con estado **Borrador**. Registrar la spec en `docs/specs/STATUS.md`; si el índice no existe, crearlo a partir de `docs/templates/status.md`.
2. **Objetivo:** una o dos oraciones. Agregar la **historia** solo si hay un actor real; en cambios técnicos (refactor, infraestructura, performance) se omite.
3. **Alcance:** "Incluye" y un "No incluye" explícito con lo que podría asumirse y queda afuera.
4. **Criterios de aceptación**, en tres grupos:
   - **Comportamiento principal:** el camino feliz de cada funcionalidad pedida.
   - **Errores y casos borde:** entradas inválidas, recurso inexistente, permisos, duplicados, límites, concurrencia cuando aplique.
   - **Regresión:** por cada dependencia hacia adentro del mapa que el cambio puede romper, el comportamiento que debe mantenerse. Si una dependencia no necesita criterio, justificarlo en "Notas".
5. **Archivos permitidos:** tomados de "Archivos afectados" del mapa, incluidos los tests. Cada archivo indica qué criterios cubre. Si se agrega un archivo que no está en el mapa, justificarlo en "Notas".

## Control de tamaño

Si la spec supera **8 criterios** o **15 archivos permitidos**, detenerse y proponer cómo dividir el cambio en varias specs independientes.

## Autocontrol antes de terminar

- [ ] Cada criterio tiene un único "Entonces" observable y se puede convertir en un test.
- [ ] Ningún criterio describe cómo implementar.
- [ ] Cada archivo permitido está vinculado al menos a un criterio.
- [ ] Cada dependencia hacia adentro del mapa tiene un criterio de regresión o una justificación en "Notas".
- [ ] "No incluye" tiene al menos un elemento.
- [ ] No quedan ambigüedades bloqueantes sin respuesta.

## Cierre

Responder con un resumen breve: cantidad de criterios por grupo, archivos permitidos, dudas que quedaron en "Notas" y la ruta de la spec. Después **esperar la aprobación**.

Cuando la persona aprueba:

1. Cambiar el estado de la spec a **Aprobada** y actualizar su fila en `docs/specs/STATUS.md`.
2. **Continuar de inmediato** delegando en el agente `desarrollador` para ejecutar `sdd-implementar` con la ruta de la spec. No pedir otra confirmación ni esperar a que la persona escriba `/implementar`.

Si en lugar de aprobar la persona pide cambios, ajustar la spec y volver a pedir aprobación.
