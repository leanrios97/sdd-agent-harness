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
| v0.1 | Núcleo mínimo: contrato, plantilla de spec, flujo guiado y agente desarrollador | 🟨 En diseño |
| v0.2 | Memoria y estado: engram, índice de specs, ciclo de vida | ⬜ |
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
- [ ] Los entregables existen y funcionan en Claude Code y en OpenCode
- [ ] Se completó al menos una spec real de punta a punta con el flujo
- [ ] README actualizado con instrucciones de uso
