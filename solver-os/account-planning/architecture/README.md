# Arquitectura — Account Planning con IA

Documentacion tecnica de arquitectura de la solucion.

## Documentos

| # | Documento | Contenido |
|---|----------|----------|
| 01 | [Solution Architecture](01-SOLUTION-ARCHITECTURE.md) | Vision general, stack tecnologico, componentes, decisiones de arquitectura, seguridad, observabilidad |
| 02 | [Sequence Flows](02-SEQUENCE-FLOWS.md) | Diagramas de secuencia para los 5 flujos principales, tiempos estimados, prompt template para Claude |
| 03 | [Data Model](03-DATA-MODEL.md) | Diagrama ER, tablas, enums, JSONB structures, indices |
| 04 | [Deployment & Infra](04-DEPLOYMENT-AND-INFRA.md) | Arquitectura de deployment, Docker Compose, estructura de repo, CI/CD pipeline, escalabilidad |
| 05 | [UI Wireframes](05-UI-WIREFRAMES.md) | Wireframes ASCII de las 4 pantallas principales: dashboard, plan de cuenta, wizard, pipeline |
| 06 | [Assumptions](06-ASSUMPTIONS.md) | Supuestos de negocio, tecnicos, datos, IA, equipo, alcance, riesgos y checklist de validacion |

## Stack resumen

| Capa | Tecnologia |
|------|-----------|
| Frontend | Angular 18 + TypeScript + Angular Material + Tailwind |
| Backend | .NET 8 (ASP.NET Core) + C# 12 + EF Core 8 + MediatR |
| AI Engine | Python + FastAPI + Claude API (Sonnet/Haiku) |
| Database | SQL Server 2022 / Azure SQL + Redis 7 |
| Infra | Docker + Terraform + GitHub Actions / Azure DevOps |
| Observabilidad | OpenTelemetry + Grafana + Serilog (alt: Application Insights) |

## Orden de lectura

1. **01-Solution Architecture** → entender el big picture
2. **03-Data Model** → entender las entidades
3. **02-Sequence Flows** → entender como interactuan
4. **05-UI Wireframes** → entender la experiencia de usuario
5. **04-Deployment** → entender como se despliega y escala
