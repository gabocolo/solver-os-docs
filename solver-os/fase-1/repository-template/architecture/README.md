# /architecture — Decisiones y Principios de Arquitectura

Contiene todas las decisiones arquitectonicas, principios transversales y diagramas C4 del proyecto.

## Contenido

| Carpeta | Proposito |
|---------|-----------|
| adrs/ | Architecture Decision Records — decisiones tecnicas documentadas y trazables |
| principles/ | Principios de arquitectura transversales (reglas que aplican a todo el sistema) |
| c4/ | Diagramas C4 en Mermaid (contexto, contenedores, componentes) |

## ADRs

Cada ADR sigue el template ADR-000-TEMPLATE.md y debe incluir:
- Contexto y problema
- Opciones evaluadas (minimo 2)
- Decision tomada con justificacion
- Consecuencias positivas y negativas
- Siguientes pasos

> Un ADR puede invocar a otro ADR — eso genera un mapa estructural de decisiones.

## Diagramas C4

Todos los diagramas deben estar en **Mermaid** (no ASCII, no imagenes). Mermaid optimiza tokens para la IA.

| Nivel | Archivo | Que muestra |
|-------|---------|-------------|
| C1 - Contexto | c4/c1-context.md | Sistema en relacion con usuarios y sistemas externos |
| C2 - Contenedores | c4/c2-containers.md | Frontend, backend, base de datos, servicios externos |
| C3 - Componentes | c4/c3-components.md | Modulos internos del backend y frontend (opcional) |

> Esta carpeta es gobernada: cambios requieren PR y revision del arquitecto.
