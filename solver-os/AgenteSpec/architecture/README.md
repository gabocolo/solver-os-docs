# Arquitectura — Spec Agent Portal

Indice de documentos de arquitectura del Spec Agent Portal.

## Documentos

| # | Documento | Descripcion |
|---|-----------|-------------|
| 01 | [Solution Architecture](01-SOLUTION-ARCHITECTURE.md) | Vision, hexagonal, componentes, seguridad |
| 02 | [Sequence Flows](02-SEQUENCE-FLOWS.md) | Diagramas de secuencia (7 flujos principales) |
| 03 | [Data Model](03-DATA-MODEL.md) | ER, tablas, enums, indices, ADR-006 |
| 04 | [Deployment & Infra](04-DEPLOYMENT-AND-INFRA.md) | Docker Compose, CI/CD, estructura de repositorio |
| 05 | [UI Wireframes](05-UI-WIREFRAMES.md) | Wireframes ASCII (6 pantallas principales) |
| 06 | [Assumptions](06-ASSUMPTIONS.md) | Supuestos, riesgos, checklist de arranque |

## Stack tecnologico

| Capa | Tecnologia |
|------|-----------|
| Frontend | Angular 21 + TypeScript + Angular Material + Tailwind CSS |
| Editor | Monaco Editor (ngx-monaco-editor) |
| Backend | .NET 10 + C# + Minimal APIs + Semantic Kernel |
| Database | PostgreSQL 16 (JSONB para specs) |
| Cache | Redis 7 |
| Messaging | Azure Service Bus |
| Auth | OAuth 2.0 (GitHub / Azure AD) |
| LLM | Claude API (Anthropic) via Semantic Kernel |
| Observabilidad | OpenTelemetry + Seq |
| CI/CD | Azure DevOps Pipelines |
| Deployment | Docker Compose (MVP) |

## Principios de arquitectura aplicados

- **ADR-001**: Hexagonal (Ports & Adapters) — dominio depende de puertos, no de adaptadores
- **ADR-002**: Idempotencia obligatoria en operaciones criticas
- **ADR-003**: Trazabilidad con correlationId en toda operacion
- **ADR-004**: Dependencias gestionadas — solo las autorizadas en la spec
- **ADR-005**: Testing strategy — TDD con GWT
- **ADR-006**: Clasificacion de datos y DLP — Privacy by Design
