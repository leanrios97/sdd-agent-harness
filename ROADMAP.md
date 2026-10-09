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

Un flujo SDD que se pueda usar desde el primer día, rápido y sin ceremonias: el agente principal guía el flujo, un agente desarrollador implementa con buenas prácticas y la persona aprueba entre cada paso.

### Flujo

```
/spec ──► aprobación ──► /implementar ──► aprobación ──► /verificar
  │                          │                               │
crea la spec            agente desarrollador:         comprueba cada
en docs/specs/          TDD estricto +                criterio de
                        estándares de desarrollo      aceptación
```

### Entregables

```
sdd-agent-harness/
├── AGENTS.md                   # Contrato único con las reglas de trabajo
├── CLAUDE.md                   # Adaptador de Claude Code (referencia a AGENTS.md)
├── .claude/agents/
│   └── desarrollador.md        # Agente desarrollador para Claude Code
├── .claude/commands/
│   ├── spec.md
│   ├── implementar.md
│   └── verificar.md
├── .opencode/                  # Agente y comandos equivalentes para OpenCode
│   └── ...                     # Ubicación y formato a confirmar (ver Preguntas abiertas)
└── docs/
    ├── estandares/
    │   └── desarrollo.md       # Buenas prácticas que aplica el agente desarrollador
    ├── specs/                  # Una spec por archivo
    └── templates/
        └── spec.md             # Plantilla de spec
```

### Contrato (`AGENTS.md`)

Reglas mínimas que valen para los dos agentes:

- Trabajar siempre a partir de una spec aprobada.
- TDD estricto: escribir primero un test que falle, después la implementación.
- Tocar solo los archivos declarados en la spec; si hace falta otro, frenar y avisar.
- Detenerse al final de cada paso y esperar aprobación.
- Conventional commits en español, sin atribución de IA.

### Agente desarrollador

Es quien ejecuta `/implementar`. No agrega un paso extra al flujo: reemplaza al agente principal durante la implementación, con contexto limpio y reglas de desarrollo propias.

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
| `/spec <descripción>` | Descripción del cambio | Spec nueva en `docs/specs/` a partir de la plantilla |
| `/implementar <spec>` | Ruta o ID de la spec | Tests y código que cumplen la spec, siguiendo TDD |
| `/verificar <spec>` | Ruta o ID de la spec | Reporte criterio por criterio: cumple / no cumple, con evidencia |

### Fuera de alcance en v0.1

- Orquestador y subagentes adicionales al desarrollador
- Memoria persistente (engram)
- Índice de specs y estados
- Validadores de CI
- Lentes de revisión
- Métricas

### Preguntas abiertas

- [ ] ¿Cómo carga OpenCode `AGENTS.md` y qué orden de precedencia tiene frente a otros archivos de instrucciones?
- [ ] ¿Dónde y en qué formato espera OpenCode los comandos personalizados?
- [ ] ¿Cómo se define un agente en OpenCode y cómo se invoca desde un comando, de forma equivalente a `.claude/agents/` en Claude Code?
- [ ] ¿Cuál es la forma recomendada en Claude Code para que `CLAUDE.md` reutilice `AGENTS.md` sin duplicar contenido?
- [ ] ¿Se puede escribir cada comando una sola vez y compartirlo entre las dos herramientas, o hace falta un archivo por herramienta?

### Criterio de terminado

- [ ] Las preguntas abiertas tienen respuesta documentada
- [ ] Los entregables existen y funcionan en Claude Code y en OpenCode
- [ ] Se completó al menos una spec real de punta a punta con el flujo
- [ ] README actualizado con instrucciones de uso
