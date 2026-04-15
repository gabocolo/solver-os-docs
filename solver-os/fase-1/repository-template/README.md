# Repositorio AI-Ready — Template

Estructura estandar para proyectos desarrollados con el framework AI-Driven Engineering (Solver OS).

> La documentacion vive en el repositorio de codigo, NO en la wiki. La wiki sirve para humanos, el repo para la IA.

## Estructura

```
/repository
  /architecture      -> ADRs, C4, principios
  /specifications    -> ai-executable-specs, use-cases
  /contracts         -> openapi / graphql / eventos
  /security          -> threat-models, politicas
  /tests             -> unit / integration / contract
  /src               -> codigo de producto
  /observability     -> logs / metrics / tracing
```

## Descripcion de carpetas

| Carpeta | Que contiene | Quien la gobierna |
|---------|-------------|------------------|
| /architecture | ADRs, diagramas C4 (Mermaid), principios de arquitectura | Arquitecto / Lider tecnico |
| /specifications | Specs L1 (dominio), L2 (sistema), L3 (cambios), contexto inicial | Arquitecto + Dev Senior |
| /contracts | Contratos OpenAPI 3.1, schemas de eventos de dominio | Arquitecto / Lider tecnico |
| /security | Modelos de amenazas, politicas de seguridad, configuracion DLP | Arquitecto + Seguridad |
| /tests | Tests unitarios (GWT), integracion, contrato, E2E, fixtures sinteticos | QA + Dev Senior |
| /src | Codigo fuente del producto (backend y/o frontend) | Dev Senior |
| /observability | Configuracion de logs, metricas, trazas, alertas | Dev Senior + DevOps |

## Archivos de contexto IA

| Archivo | Donde va | Proposito |
|---------|----------|-----------|
| CLAUDE.md | Raiz del repo | Contexto general del framework |
| CLAUDE-BACKEND.md | Raiz del repo backend | Contexto IA especifico para backend |
| CLAUDE-FRONTEND.md | Raiz del repo frontend | Contexto IA especifico para frontend |

> Cada repositorio (backend, frontend) tiene su propio CLAUDE.md con contexto aislado. Las decisiones tecnicas de back son diferentes a las de front (recomendacion de Osvaldo Arias).

## Flujo de trabajo

```
1. Llenar CONTEXT-INTAKE-TEMPLATE.md (captura de contexto inicial)
2. Generar specs L1 (dominio) con el agente → /specifications/domain/
3. Generar specs L2 (sistema) desde L1 aprobada → /specifications/system/
4. Generar contratos OpenAPI con SK-003 → /contracts/openapi/
5. Generar tests GWT con SK-002 → /tests/unit/
6. Generar codigo con prompts BE + FE en paralelo → /src/
7. Validar con pipeline (SAST + SCA + DLP + tests) → Gate 2
8. Revision humana → Gate 3
9. Merge y deploy
```

## Quality Gates

| Gate | Que valida | Bloqueante |
|------|-----------|------------|
| Gate 1 | Spec aprobada + clasificacion datos + ADRs | Si |
| Gate 2 | Tests + SAST + SCA + DLP Scan | Si |
| Gate 3 | Revision senior + proteccion datos | Si |

## Reglas del repositorio

- Diagramas siempre en **Mermaid** (no ASCII, no imagenes) — optimiza tokens
- PRs < 400 lineas — dividir si es mayor
- Datos sinteticos siempre — nunca PII real en codigo, tests ni fixtures
- El prompt invoca la spec, NO contiene reglas de negocio
- Backend y frontend son dominios separados con contexto aislado
- Los contratos OpenAPI son donde convergen back y front
- Toda documentacion versionada con semver

---

_Template del framework AI-Driven Engineering v1.1 — Tech&Solve (Solver OS)_
