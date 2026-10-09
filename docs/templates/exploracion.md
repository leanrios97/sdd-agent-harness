# <ID> — Mapa de impacto: <título del cambio>

- **Fecha:** AAAA-MM-DD
- **Pedido:** descripción del cambio tal como se solicitó.

## Puntos de entrada

Símbolos, archivos, rutas o tablas que nombra el pedido.

| Elemento | Ubicación |
|----------|-----------|
| ... | `ruta:línea` |

## Relaciones

Cada categoría se completa siempre. Si no hay hallazgos, escribir "revisado: ninguno" e indicar cómo se buscó.

### Dependencias hacia afuera (qué usa)
- `ruta:línea` — descripción

### Dependencias hacia adentro (quién lo usa)
- `ruta:línea` — descripción

### Referencias indirectas
Configuración, rutas HTTP, eventos o colas, claves de caché, plantillas, inyección de dependencias.
- `ruta:línea` — descripción

### Datos
Modelos, esquemas, migraciones, contratos de API.
- `ruta:línea` — descripción

### Tests
- `ruta` — qué cubre

### Documentación
- `ruta` — qué describe

## Búsquedas realizadas

Términos y patrones buscados, para que se pueda auditar la cobertura.

| Búsqueda | Resultados |
|----------|------------|
| `patrón` | N |

## Archivos afectados

Propuesta para la sección "Archivos permitidos" de la spec.

| Archivo | Motivo |
|---------|--------|
| `ruta` | crear / modificar — por qué |

## Riesgos y dudas

- ...
