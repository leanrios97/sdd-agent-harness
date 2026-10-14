# Roadmap

## Principio

Empezar simple y rápido. Sumar complejidad solo cuando el uso real demuestre que hace falta.

Cada versión tiene que poder usarse de punta a punta antes de pasar a la siguiente. Si una pieza no reduce errores ni ahorra tiempo de forma medible, no entra.

## Alcance

- **Herramientas soportadas:** Claude Code y OpenCode.
- **Única dependencia externa:** engram (memoria persistente), a partir de v0.2.
- **Idioma:** toda la documentación, specs y mensajes de commit en español; identificadores de código en inglés.

## Versiones

| Versión | Objetivo | Estado |
|---------|----------|--------|
| v0.1 | Núcleo mínimo: contrato, plantilla de spec, flujo guiado y agentes explorador y desarrollador | 🟩 Validada en Claude Code y OpenCode |
| v0.2 | Memoria y estado: engram, índice de specs, ciclo de vida | 🟨 En diseño |
| v0.3 | Roles y controles: subagentes solo donde aporten, revisión antes de integrar, control de alcance | ⬜ |
| v0.4 | Calidad automatizada: validadores de CI, lentes de revisión, vía rápida | ⬜ |
| v0.5 | Métricas: tokens, costo, retrabajo y lead time por spec | ⬜ |

---

## v0.1 — Núcleo mínimo

### Objetivo

Un flujo SDD que se pueda usar desde el primer día, rápido y sin ceremonias: el agente principal guía el flujo, un agente explorador mapea todo lo que el cambio afecta, un agente desarrollador implementa con buenas prácticas y la persona aprueba entre cada paso.

### Flujo

```
/explorar ──► /spec ──► /implementar ──► /verificar
    │           │             │              │
 agente      crea la     agente          comprueba cada
 explorador: spec a      desarrollador:  criterio de
 mapa de     partir del  TDD estricto +  aceptación
 impacto     mapa        estándares

        (aprobación de la persona entre cada paso)
```

### Entregables

```
sdd-agent-harness/
├── AGENTS.md                   # Contrato único con las reglas de trabajo (fuente única)
├── CLAUDE.md                   # Solo importa: @AGENTS.md y @docs/estandares/desarrollo.md
├── opencode.json               # "instructions": ["docs/estandares/desarrollo.md"]
├── .claude/
│   ├── skills/                 # Lógica de cada fase, compartida por ambas herramientas
│   │   ├── sdd-explorar/SKILL.md
│   │   ├── sdd-spec/SKILL.md
│   │   ├── sdd-implementar/SKILL.md
│   │   └── sdd-verificar/SKILL.md
│   ├── agents/
│   │   ├── explorador.md       # Agente explorador, solo lectura (formato Claude Code)
│   │   └── desarrollador.md    # Agente desarrollador (formato Claude Code)
│   └── commands/               # Envoltorios finos que invocan cada skill
│       ├── explorar.md         # context: fork + agent: explorador
│       ├── spec.md
│       ├── implementar.md      # context: fork + agent: desarrollador
│       └── verificar.md
├── .opencode/
│   ├── agents/
│   │   ├── explorador.md       # Agente explorador, solo lectura (formato OpenCode, mode: subagent)
│   │   └── desarrollador.md    # Agente desarrollador (formato OpenCode, mode: subagent)
│   └── commands/               # Envoltorios finos que invocan cada skill
│       ├── explorar.md         # agent: explorador + subtask: true
│       ├── spec.md
│       ├── implementar.md      # agent: desarrollador + subtask: true
│       └── verificar.md
└── docs/
    ├── estandares/
    │   └── desarrollo.md       # Buenas prácticas que aplica el agente desarrollador
    ├── exploraciones/          # Un mapa de impacto por cambio
    ├── specs/                  # Una spec por archivo
    └── templates/
        ├── exploracion.md      # Plantilla del mapa de impacto
        └── spec.md             # Plantilla de spec
```

### Contrato (`AGENTS.md`)

Reglas mínimas que valen para los dos agentes:

- Trabajar siempre a partir de una spec aprobada.
- TDD estricto: escribir primero un test que falle, después la implementación.
- Tocar solo los archivos declarados en la spec; si hace falta otro, frenar y avisar.
- Detenerse al final de cada paso y esperar aprobación.
- Conventional commits en español, sin atribución de IA.

### Agente explorador

Es quien ejecuta `/explorar`. Trabaja en solo lectura y con contexto propio, así la lectura masiva de archivos no satura la conversación principal.

Su objetivo es que **ninguna relación del código quede sin revisar**. Para eso no explora libremente: recorre un método fijo y deja constancia de cada paso.

1. **Puntos de entrada:** identifica los símbolos, archivos, rutas o tablas que nombra el pedido.
2. **Dependencias hacia afuera:** qué usa cada punto de entrada (imports, llamadas, servicios externos).
3. **Dependencias hacia adentro:** quién usa cada punto de entrada (búsqueda de todas las referencias, no solo de los imports directos).
4. **Referencias indirectas:** usos por texto que un análisis de imports no detecta: nombres en configuración, rutas HTTP, nombres de eventos o colas, claves de caché, cadenas en plantillas, inyección de dependencias.
5. **Datos:** modelos, esquemas, migraciones y contratos de API afectados.
6. **Tests y documentación:** tests que cubren lo afectado y documentos que lo describen.
7. **Iteración:** cada archivo nuevo encontrado vuelve a pasar por los pasos 2 a 6, hasta que no aparecen relaciones nuevas.

El resultado es un **mapa de impacto** en `docs/exploraciones/`, a partir de `docs/templates/exploracion.md`. Cada categoría se marca siempre, aunque esté vacía ("revisado: ninguno"), para que una omisión sea visible y no se confunda con algo que no se buscó. El mapa alimenta la sección "Archivos permitidos" de la spec.

### Agente desarrollador

Es quien ejecuta `/implementar`. No agrega un paso extra al flujo: reemplaza al agente principal durante la implementación, con contexto limpio y reglas de desarrollo propias. Como no ve el historial de la conversación, la spec es su única fuente de verdad.

Sus reglas viven en `docs/estandares/desarrollo.md`, una única fuente que después reutilizan `/verificar` y las revisiones de versiones futuras:

- **Programación orientada a objetos** con responsabilidades claras y encapsulamiento.
- **SOLID** como criterio de diseño, aplicado de forma pragmática.
- **Patrones de diseño** solo cuando resuelven un problema presente en la spec; nunca por anticipado.
- **Arquitectura** limpia/hexagonal: el dominio no depende de frameworks ni de infraestructura.
- **KISS y YAGNI** como contrapeso: la solución más simple que cumpla los criterios de aceptación.
- **Nombres expresivos**, funciones cortas y sin duplicación.
- **TDD estricto:** test que falla → implementación mínima → refactor.

### Plantilla de spec

| Sección | Contenido |
|---------|-----------|
| Objetivo | Qué problema resuelve, en una o dos oraciones |
| Alcance | Qué entra y qué queda afuera explícitamente |
| Criterios de aceptación | Lista verificable; cada criterio se traduce en al menos un test |
| Archivos permitidos | Rutas que la implementación puede crear o modificar |
| Notas | Decisiones, dudas o dependencias |

### Comandos

| Comando | Entrada | Resultado |
|---------|---------|-----------|
| `/explorar <descripción>` | Descripción del cambio | Mapa de impacto en `docs/exploraciones/` |
| `/spec <exploración>` | Ruta o ID del mapa de impacto | Spec nueva en `docs/specs/` a partir de la plantilla y del mapa |
| `/implementar <spec>` | Ruta o ID de la spec | Tests y código que cumplen la spec, siguiendo TDD |
| `/verificar <spec>` | Ruta o ID de la spec | Reporte criterio por criterio: cumple / no cumple, con evidencia |

### Fuera de alcance en v0.1

- Orquestador y subagentes adicionales al explorador y al desarrollador
- Memoria persistente (engram)
- Índice de specs y estados
- Validadores de CI
- Lentes de revisión
- Métricas

### Compatibilidad entre Claude Code y OpenCode

Respuestas obtenidas de la documentación oficial de cada herramienta.

**¿Cómo carga OpenCode `AGENTS.md`?**
Lee `AGENTS.md` de la raíz del proyecto (y el global en `~/.config/opencode/`). Usa `CLAUDE.md` solo como respaldo si no existe `AGENTS.md`. No interpreta referencias `@archivo` dentro de `AGENTS.md`; los archivos extra se suman con el campo `instructions` de `opencode.json`.
Fuentes: [Rules](https://opencode.ai/docs/rules/), [Config](https://opencode.ai/docs/config/)

**¿Cómo reutiliza Claude Code `AGENTS.md`?**
La forma recomendada es un `CLAUDE.md` que importe `@AGENTS.md`. Las versiones recientes leen `AGENTS.md` solas, pero solo si no hay `CLAUDE.md` y no en todos los casos, así que el import es más robusto. En Windows se recomienda el import en lugar de un enlace simbólico. Los imports admiten hasta 4 niveles de anidamiento.
Fuente: [Memory](https://code.claude.com/docs/en/memory)

**¿Cómo se definen los comandos?**
- OpenCode: Markdown en `.opencode/commands/`, con `description`, `agent`, `model` y `subtask` en el frontmatter.
- Claude Code: los comandos se unificaron con las skills. `.claude/commands/x.md` y `.claude/skills/x/SKILL.md` crean el mismo `/x`.

Fuentes: [OpenCode Commands](https://opencode.ai/docs/commands/), [Claude Code Skills](https://code.claude.com/docs/en/skills)

**¿Cómo se define el agente desarrollador y cómo se invoca desde un comando?**
- OpenCode: `.opencode/agents/desarrollador.md` con `mode: subagent` y `permission`; el comando lo fuerza con `agent` + `subtask: true`.
- Claude Code: `.claude/agents/desarrollador.md` con `name`, `description`, `tools` y `model`; el comando lo fuerza con `context: fork` + `agent`. El subagente no ve el historial de la conversación, así que la spec debe pasarse completa.

Fuentes: [OpenCode Agents](https://opencode.ai/docs/agents/), [Claude Code Subagents](https://code.claude.com/docs/en/sub-agents)

**¿Qué se puede escribir una sola vez?**
- `AGENTS.md` (lo leen ambas herramientas).
- Las skills en `.claude/skills/<nombre>/SKILL.md`: OpenCode también las lee, aunque como conocimiento que el modelo carga bajo demanda, no como comandos `/`.

**¿Qué hay que duplicar?**
- **Los agentes explorador y desarrollador:** los formatos de frontmatter son incompatibles. Ambos archivos deben ser cortos y remitir a `AGENTS.md` y a la skill.
- **Los comandos:** OpenCode no documenta que lea `.claude/commands/`, y la forma de forzar el agente es distinta. Cada comando es un envoltorio de pocas líneas que invoca la skill correspondiente.
- **La referencia a los estándares:** `@` en `CLAUDE.md` para Claude Code y `instructions` en `opencode.json` para OpenCode.

**Regla de argumentos:** usar solo `$ARGUMENTS`, nunca posicionales. Claude Code numera desde `$0` y OpenCode desde `$1`.

### Criterio de terminado

- [x] Las preguntas de compatibilidad tienen respuesta documentada
- [x] Los entregables existen y funcionan en Claude Code
- [x] Los entregables funcionan en OpenCode
- [x] Se completó una spec real de punta a punta con el flujo (proyecto de prueba descartable)
- [x] README actualizado con instrucciones de uso

### Resultado de la validación en Claude Code

Prueba sobre un proyecto descartable con trampas deliberadas. El flujo completo (`/explorar`, `/spec`, `/implementar`, `/verificar`) funcionó y detectó las cuatro:

| Trampa | Resultado |
|--------|-----------|
| Referencia indirecta (nombre de función solo en un YAML, resuelto con importación dinámica) | Detectada por el explorador |
| Mismo identificador con variantes de nombre (`user_id` y `userId`) | Detectada y planteada como duda |
| Test que ya falla antes de empezar | Identificado como preexistente y no modificado |
| Archivo que usa lo afectado pero no se ve impactado | Descartado y excluido de los archivos permitidos |

### Resultado de la validación en OpenCode

El mismo proyecto de prueba, restaurado a su estado inicial, se recorrió en una sesión real de OpenCode con el flujo completo, incluida la continuidad automática entre pasos. El flujo funcionó correctamente.

### Mejoras pendientes

Surgieron de la validación y no bloquean v0.1:

- **Commit base en la spec.** Registrar el commit de partida al empezar a implementar para que `/verificar` compare contra él. Hoy depende de que los cambios estén sin commitear.
- **Ejecución en primer plano.** Los comandos que corren en un subagente lo hacen en segundo plano por defecto; evaluar `background: false` para que el resultado se muestre al terminar.
- **Instalador.** Incorporar el harness a un proyecto exige cinco pasos manuales (ver README). Automatizarlos con un script de bootstrap.
- **Permisos de bash del desarrollador en OpenCode.** Con `"*": ask` pide confirmación por cada comando; evaluar permitir de forma explícita el comando de tests de cada proyecto.

---

## v0.2 — Memoria y estado

### Objetivo

Que el harness recuerde entre sesiones qué está en curso, qué se decidió y qué se descubrió, sin duplicar lo que ya vive en el repositorio.

### Problema

Hoy el estado de cada spec está solo en su propio encabezado y no hay una vista de conjunto. Al abrir una sesión nueva, el agente no sabe qué quedó en curso, qué decisiones se tomaron ni qué encontró el explorador en cambios anteriores.

### Modo de trabajo

Un único modo: **spec + memoria**. Las specs son la fuente de verdad y viven en el repositorio; la memoria (engram) es aditiva y guarda el conocimiento que sirve para otros cambios. No hay selección de modo al iniciar un proyecto. Una línea en `AGENTS.md` (`Memoria: engram`) permite desactivarla.

### Piezas

| # | Pieza | Descripción | Problema que resuelve |
|---|-------|-------------|------------------------|
| 1 | Índice `docs/specs/STATUS.md` | Tabla con una fila por spec: ID, título, estado, enlaces a su exploración y verificación, fecha. Las skills la actualizan en cada transición | No hay vista de conjunto |
| 2 | Ciclo de vida formal | Estados y transiciones válidas, definidos en `AGENTS.md` | Los estados son texto libre y se pueden saltear |
| 3 | Commit base en la spec | Al empezar a implementar se registra el commit de partida; `/verificar` compara contra él | El diff queda vacío si se commitea a mitad de camino |
| 4 | Comando `/estado` | Lee el índice e informa qué está en curso y cuál es el próximo paso | Retomar el trabajo en una sesión nueva |
| 5 | Memoria con engram | Guarda decisiones, hallazgos y desvíos entre sesiones | El conocimiento de un cambio se pierde para el siguiente |

### Ciclo de vida de una spec

| Estado | Se llega cuando | Quién lo cambia |
|--------|-----------------|-----------------|
| Borrador | Se escribe la spec | `/spec` |
| Aprobada | La persona aprueba | Agente principal |
| Implementada | La implementación termina | `/implementar` |
| Verificada | La verificación da "Aprobada" o "Con observaciones" | `/verificar` |
| Bloqueada | Falta una decisión para avanzar; vuelve al estado anterior al resolverse | Agente principal |
| Descartada | La persona decide no continuar (solo desde Borrador o Aprobada) | Agente principal |

- Una verificación "Rechazada" deja la spec en Implementada hasta que se corrija.
- Si una spec aprobada cambia, vuelve a Borrador, requiere una nueva aprobación y el motivo se registra en memoria.

### Memoria

**Qué va en cada lugar**

| Dónde | Qué guarda | Por qué |
|-------|------------|---------|
| Archivos del repositorio | Specs, mapas de impacto, verificaciones y estado | Fuente de verdad, versionada y visible |
| Engram | Decisiones transversales, hallazgos sobre la arquitectura, gotchas y desvíos | Conocimiento que sirve para *otros* cambios |

**Qué se guarda**

| Momento | Contenido |
|---------|-----------|
| Se aprueba una spec | La decisión y su motivo, incluidas las dudas bloqueantes y cómo se resolvieron |
| Una spec aprobada cambia | Qué cambió y por qué; actualiza el mismo registro |
| El explorador encuentra algo no obvio | El hallazgo y dónde está |
| La verificación da observaciones o rechaza | Qué falló y la causa |
| Hay un desvío o se frena una implementación | Qué faltaba en la spec |

No se guarda: avance rutinario, contenido de archivos ni nada que pueda leerse en el repositorio o en git.

**Cómo funciona**

- **Punto de guardado:** la aprobación de la persona. Los subagentes tienen una lista cerrada de herramientas y no ven la conversación, así que quien escribe en memoria es el agente principal, antes de iniciar el paso siguiente.
- **Clave estable por spec** (`<proyecto>/spec/<ID>`): un cambio posterior actualiza el registro en lugar de duplicarlo.
- **Lectura:** antes de explorar, el agente principal consulta la memoria y pasa al explorador lo relevante (a confirmar según lo que permita cada herramienta).

### Entregables

```
docs/specs/STATUS.md                      # Índice de specs
.claude/skills/sdd-estado/SKILL.md        # Lógica de /estado, compartida
.claude/commands/estado.md
.opencode/commands/estado.md
AGENTS.md                                 # + ciclo de vida, + memoria
docs/templates/spec.md                    # + campo "Commit base"
.claude/skills/sdd-*/SKILL.md             # actualizadas: índice, commit base y puntos de guardado
```

### Orden de trabajo

1. Índice, ciclo de vida y commit base (sin dependencias externas).
2. Comando `/estado`.
3. Memoria con engram, al final porque requiere investigar la integración en cada herramienta.

### Fuera de alcance

- Subagentes adicionales y orquestador (v0.3).
- Validadores de CI y lentes de revisión (v0.4).
- Métricas (v0.5).

### Investigación de engram

Resultados de la documentación del proyecto y de las herramientas, más una comprobación de conexión en una sesión real.

**Nombres de las herramientas**

| Instalación | Nombre en la herramienta |
|-------------|--------------------------|
| Claude Code, plugin con MCP empaquetado | `mcp__plugin_engram_engram__mem_save` |
| Claude Code, servidor MCP independiente | `mcp__engram__mem_save` |
| OpenCode | `engram_mem_save` |

El prefijo depende de cómo se instaló, así que **el harness no debe fijar nombres de herramientas de engram en las definiciones de sus agentes**. Ver la decisión de abajo.

**Subagentes y MCP**

- En Claude Code, un subagente con `tools` omitido hereda todas las herramientas, incluidas las MCP. Con una lista explícita se admiten patrones por servidor (`mcp__<servidor>__*`).
- En OpenCode, el bloque `permission` del agente admite patrones sobre el nombre de la herramienta (`engram_*`).

**Proyecto, almacenamiento y upsert**

- El proyecto se determina por: proyecto explícito, luego `--project` o la variable `ENGRAM_PROJECT`, luego el directorio de trabajo (repositorio git). Un archivo `.engram/config.json` en el repositorio fija el nombre.
- Comprobado en vivo: desde una carpeta con varios repositorios, guardar falla con `ambiguous_project`. Dentro de un único repositorio, o con `.engram/config.json`, no debería ocurrir.
- La base es una sola por máquina (`~/.engram/engram.db`) y el proyecto es un campo. Es probable que Claude Code y OpenCode la compartan, aunque la documentación no lo garantiza.
- `topic_key` hace upsert: reutiliza el último registro con el mismo proyecto, ámbito y clave, e incrementa su revisión (según la documentación; sin comprobar en vivo).

**Riesgos conocidos**

- El hook previo a herramientas del plugin de Claude Code puede denegar escrituras si no hay una sesión registrada. Llamar al MCP directamente lo evita. Falta comprobar el comportamiento real.
- `mem_update` exige indicar el proyecto esperado además del identificador.

### Decisiones

- **Los subagentes no usan engram.** Leer y escribir memoria es tarea del agente principal. Así las definiciones de los agentes no dependen del prefijo de las herramientas y el mismo contrato sirve en ambas herramientas.
- **`/explorar` deja de ejecutarse como subagente directo.** El agente principal consulta la memoria y delega en el explorador pasando lo relevante en el pedido. Alternativa descartada: darle al explorador herramientas de lectura de engram, por la dependencia del prefijo.
- **El instalador crea `.engram/config.json`** con el nombre del proyecto.

### Preguntas abiertas

- [ ] Comprobar en vivo, dentro de un repositorio, el guardado, el upsert por `topic_key` y la ausencia de bloqueos por sesión no registrada, en Claude Code y en OpenCode.
- [ ] Confirmar que ambas herramientas comparten la misma base en una máquina.

### Riesgos

- **Desincronización:** si una spec se edita a mano, el índice queda desactualizado. El validador de v0.4 lo cubre; hasta entonces depende de la disciplina.
- **Ruido en memoria:** guardar de más hace que deje de servir. Por eso los disparadores son una lista cerrada.

### Criterio de terminado

- [ ] Cerrar una sesión, abrir otra y que `/estado` informe exactamente dónde quedó el trabajo
- [ ] Una decisión tomada en una spec aparece como contexto al explorar un cambio relacionado
- [ ] El ciclo de vida se respeta en las skills: ninguna transición fuera de la tabla
- [ ] Validado en Claude Code y en OpenCode
