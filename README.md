# SDD Agent Harness

Harness reutilizable e independiente de la herramienta para desarrollo guiado por especificaciones (SDD) asistido por IA. Coordina agentes de programación en Claude Code y OpenCode mediante un contrato compartido, un pipeline de especificaciones con controles de calidad y puntos de revisión humana.

> **Estado:** v0.1. Validado en Claude Code; la validación en OpenCode está pendiente. Ver [ROADMAP.md](ROADMAP.md).

## Flujo

```
/explorar ──► /spec ──► /implementar ──► /verificar
```

| Paso | Qué hace | Quién lo ejecuta |
|------|----------|------------------|
| `/explorar <cambio>` | Construye el mapa de impacto: todas las relaciones del código que el cambio afecta, incluidas las referencias indirectas | Agente `explorador` (solo lectura) |
| `/spec <mapa>` | Escribe la spec con criterios de aceptación Dado/Cuando/Entonces, tras resolver las dudas bloqueantes | Agente principal |
| `/implementar <spec>` | Implementa con TDD estricto, solo en los archivos permitidos | Agente `desarrollador` |
| `/verificar <spec>` | Comprueba cada criterio con evidencia y controla el alcance | Agente principal |

La persona aprueba el resultado de cada paso. **Una sola aprobación inicia el paso siguiente**, sin escribir otro comando.

## Qué incluye

```
AGENTS.md                  # Contrato de trabajo (fuente única de reglas)
CLAUDE.md                  # Importa AGENTS.md y los estándares (Claude Code)
opencode.json              # Suma los estándares como instrucciones (OpenCode)
.claude/
  skills/                  # Lógica de cada paso, compartida por ambas herramientas
  agents/                  # Explorador y desarrollador (formato Claude Code)
  commands/                # Comandos del flujo
.opencode/
  agents/                  # Explorador y desarrollador (formato OpenCode)
  commands/                # Comandos del flujo
docs/
  estandares/desarrollo.md # POO, SOLID, patrones, arquitectura, KISS/YAGNI, TDD
  templates/               # Plantillas de exploración y de spec
  exploraciones/           # Mapas de impacto de cada cambio
  specs/                   # Una spec por cambio
  verificaciones/          # Reportes de verificación
  decisions/               # ADRs del harness
```

## Incorporarlo a un proyecto

Por ahora es manual. Desde la raíz de este repositorio, hacia la raíz del proyecto destino:

1. Copiar `AGENTS.md`, `CLAUDE.md` y `opencode.json`.
2. Copiar `.claude/` y `.opencode/`.
3. Copiar `docs/estandares/` y `docs/templates/`.
4. Crear las carpetas vacías `docs/exploraciones/`, `docs/specs/` y `docs/verificaciones/`.
5. Inicializar git en el proyecto si no lo está y hacer un commit inicial: `/verificar` usa `git diff` para controlar el alcance.

Después, abrir el proyecto con `claude` u `opencode` y arrancar con `/explorar`.

## Principios

- **Simple primero.** Cada pieza tiene que justificar su costo en tiempo; se suma complejidad por versión, cuando el uso real lo demuestra.
- **La spec es la fuente de verdad.** Lo que no está en la spec no se implementa.
- **Nada se pierde en la exploración.** Cada categoría de relación se revisa y queda constancia de lo que se buscó, también cuando no se encontró nada.
- **Verificación independiente.** El reporte de quien implementó es una pista; la evidencia se vuelve a comprobar.

## Decisiones de arquitectura

- [ADR-0001: No adoptar frameworks de specs](docs/decisions/ADR-0001-no-adoptar-frameworks-de-specs.md)

## Hoja de ruta

Ver [ROADMAP.md](ROADMAP.md) para las versiones, los resultados de la validación y las mejoras pendientes.
