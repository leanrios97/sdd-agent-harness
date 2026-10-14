---
name: sdd-explorar
description: Construye el mapa de impacto de un cambio recorriendo de forma sistemática todas las relaciones del código (dependencias, referencias indirectas, datos, tests y documentación). Usar antes de escribir una spec.
---

# Explorar: mapa de impacto

Objetivo: encontrar **todo** lo que el cambio afecta, y dejar constancia de cómo se buscó. Una relación que no se encontró tiene que ser visible como "buscada y no encontrada", nunca como silencio.

## Reglas

- Solo lectura. La única escritura permitida es el archivo del mapa en `docs/exploraciones/`.
- No proponer soluciones ni diseño: solo describir lo que existe y lo que el cambio toca.
- Toda afirmación lleva su ubicación (`ruta:línea`).
- No frenar en el primer nivel: seguir el método hasta que no aparezcan relaciones nuevas.

## Preparación

1. Leer el pedido completo y anotar los términos clave: nombres de clases, funciones, módulos, endpoints, tablas, campos y conceptos de negocio.
2. Para cada término, generar sus **variantes de nombre**, porque el mismo concepto aparece escrito de formas distintas:
   - `userId`, `user_id`, `UserId`, `user-id`, `USER_ID`
   - singular y plural (`order`, `orders`)
   - el término de negocio en otros idiomas si el código los mezcla
3. Identificar qué se excluye de las búsquedas (dependencias instaladas, artefactos de build, lockfiles) y registrarlo.

## Método

Mantener dos listas: **pendientes** (elementos por analizar) y **visitados**. Al inicio, pendientes = puntos de entrada.

### Paso 1 — Puntos de entrada
Ubicar en el código cada término del pedido. Buscar por nombre exacto y por variantes. Si un término no aparece en el código, registrarlo igual: puede ser algo nuevo que el cambio crea.

### Paso 2 — Para cada elemento pendiente, recorrer las seis categorías

**a. Dependencias hacia afuera (qué usa)**
- Leer el archivo: imports, clases base, llamadas a otros módulos, servicios externos, variables de entorno.

**b. Dependencias hacia adentro (quién lo usa)**
- Buscar el nombre en todo el repositorio, con límites de palabra para evitar coincidencias parciales.
- Buscar también:
  - re-exportaciones (archivos índice, `__init__.py`, barrels);
  - imports con alias (`import X as Y`) y luego los usos del alias;
  - subclases e implementaciones de la interfaz;
  - registros en decoradores, routers, contenedores de inyección o fábricas.

**c. Referencias indirectas (las que un análisis de imports no ve)**
Buscar el nombre y sus variantes **como texto** en:
- configuración (`.yaml`, `.yml`, `.json`, `.toml`, `.ini`, `.env.example`);
- rutas HTTP y nombres de endpoints;
- nombres de eventos, colas, tópicos y tareas en segundo plano;
- claves de caché y de feature flags;
- SQL escrito a mano y consultas armadas como texto;
- plantillas, serialización y nombres de campos en payloads;
- imports dinámicos y reflexión (nombres armados en tiempo de ejecución).

**d. Datos**
- Modelos, esquemas de validación, migraciones, contratos de API (OpenAPI, GraphQL), fixtures y seeds.
- Si el cambio toca un campo, seguirlo en todos esos lugares.

**e. Tests**
- Tests que importan o mencionan el elemento, sus fixtures y sus dobles de prueba.

**f. Documentación**
- Documentos, ADRs, specs anteriores y comentarios que describen el comportamiento.

### Paso 3 — Iterar
Cada archivo nuevo que aparece en las categorías a, b, c o d y que el cambio puede afectar pasa a **pendientes**, salvo que ya esté en **visitados**. Repetir el paso 2 hasta que pendientes quede vacía.

### Control de alcance
Si el mapa supera los **30 archivos afectados**, detenerse, reportar lo encontrado hasta ese momento y preguntar si el cambio debe dividirse antes de seguir.

## Resultado

1. Calcular el ID: el siguiente número de cuatro dígitos entre los archivos existentes de `docs/exploraciones/` y `docs/specs/` (`0001`, `0002`, ...).
2. Escribir `docs/exploraciones/<ID>-<slug>.md` siguiendo `docs/templates/exploracion.md`.
3. Registrar en "Búsquedas realizadas" cada patrón buscado y su cantidad de resultados.

## Autocontrol antes de terminar

- [ ] Cada categoría tiene hallazgos o "revisado: ninguno", con la búsqueda que lo respalda.
- [ ] Cada término del pedido y cada variante de nombre se buscó.
- [ ] Cada referencia hacia adentro encontrada se visitó o se justificó por qué no se afecta.
- [ ] Cada archivo de "Archivos afectados" aparece en alguna relación del mapa.
- [ ] La lista de pendientes quedó vacía.

## Cierre

Responder con un resumen breve: cantidad de archivos afectados, relaciones más relevantes, riesgos y dudas, y la ruta del mapa. Después **detenerse y esperar aprobación** antes de pasar a `/spec`.
