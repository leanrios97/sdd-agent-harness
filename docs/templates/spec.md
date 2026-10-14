# <ID> — <Título del cambio>

- **Estado:** Borrador | Aprobada | Implementada | Verificada
- **Fecha:** AAAA-MM-DD
- **Exploración:** `docs/exploraciones/<ID>-<slug>.md`

## Objetivo

Qué problema resuelve este cambio, en una o dos oraciones.

**Historia** *(opcional: solo si hay un actor real)*
Como <rol>, quiero <acción>, para <beneficio>.

## Alcance

**Incluye**
- ...

**No incluye**
- ...

## Criterios de aceptación

Cada criterio es verificable y se traduce en al menos un test. Describen qué debe pasar, nunca cómo implementarlo.

### Comportamiento principal

#### CA-1: <nombre corto>
- **Dado** <contexto inicial>
- **Cuando** <acción>
- **Entonces** <resultado esperado>

### Errores y casos borde

#### CA-2: <nombre corto>
- **Dado** ...
- **Cuando** ...
- **Entonces** ...

### Regresión

Comportamiento existente que debe seguir funcionando, tomado de las dependencias hacia adentro del mapa de impacto.

#### CA-3: <nombre corto>
- **Dado** ...
- **Cuando** ...
- **Entonces** ...

## Archivos permitidos

Rutas que la implementación puede crear o modificar. Cualquier otro archivo requiere aprobación.

| Archivo | Acción | Criterios |
|---------|--------|-----------|
| `ruta/al/archivo` | crear / modificar | CA-1, CA-2 |

## Notas

Decisiones tomadas, dudas no bloqueantes y dependencias.
