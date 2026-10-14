---
name: sdd-estado
description: Informa en qué punto quedó el trabajo a partir del índice de specs: qué está en curso, qué está bloqueado y cuál es el próximo paso. Usar al retomar una sesión o para tener una vista de conjunto.
---

# Estado: dónde quedó el trabajo

Objetivo: dar en pocas líneas la situación del proyecto y el próximo paso concreto. Es solo lectura: no modifica ningún archivo.

## Paso 1 — Leer el índice

1. Leer `docs/specs/STATUS.md`.
2. Si no existe, responder "No hay specs registradas" y sugerir `/explorar` para empezar un cambio.

## Paso 2 — Contrastar con la realidad

Para cada spec del índice:

1. Leer el estado del encabezado de la spec y compararlo con el del índice.
2. Comprobar que existan los archivos enlazados (exploración, spec y, si corresponde, verificación).
3. Registrar cada diferencia como **inconsistencia**, indicando qué dice cada fuente. No corregirla.

Además, ejecutar `git status` y anotar si hay cambios sin commitear.

## Paso 3 — Clasificar

- **En curso:** Borrador, Aprobada o Implementada.
- **Bloqueadas:** indicar qué decisión falta, tomada de las "Notas" de la spec.
- **Cerradas:** Verificada o Descartada; solo se cuentan.

## Paso 4 — Próximo paso

| Estado | Próximo paso |
|--------|--------------|
| Borrador | Revisar y aprobar la spec |
| Aprobada | Implementar con `/implementar` |
| Implementada | Verificar con `/verificar`; si la última verificación fue rechazada, corregir con `/implementar` |
| Bloqueada | Resolver la decisión pendiente |
| Verificada | Commitear si todavía hay cambios sin commitear |

## Cierre

Responder con este formato, sin agregar secciones vacías:

1. **En curso:** una línea por spec, con ID, título, estado y próximo paso.
2. **Bloqueadas:** una línea por spec con la decisión que falta.
3. **Inconsistencias:** diferencias entre el índice y las specs.
4. **Cambios sin commitear:** sí o no.
5. **Próximo paso sugerido:** una sola acción, la de la spec más avanzada en el flujo.
