# Diseno de acelerador transversal

| Campo | Valor |
|-------|-------|
| **Nombre** | Spec Agent Portal — Acelerador de Gobernanza IA |
| **Version** | 1.0.0 |
| **Owner** | Equipo de Arquitectura — Tech&Solve |
| **Fecha** | 2026-04-09 |
| **Estado** | DRAFT |

---

## Problema que resuelve

Los equipos de desarrollo que adoptan IA generativa enfrentan:
- **Sin estructura**: cada desarrollador usa IA de forma distinta, sin gobernanza
- **Sin trazabilidad**: no hay conexion entre la decision de negocio, la spec y el codigo generado
- **Retrabajo alto**: codigo generado sin especificacion produce deuda tecnica
- **Riesgo de PII**: datos sensibles se filtran en prompts, logs y fixtures
- **Metricas ausentes**: no hay visibilidad del impacto real de la IA en la calidad

El Spec Agent Portal elimina estos problemas operacionalizando el framework AI-Driven Engineering como una plataforma web con gobernanza automatizada.

---

## Alcance

### Que hace
- Portal web para gestionar el ciclo completo de especificaciones (L1/L2/L3)
- Generacion asistida por IA con seleccion automatica de modelo (Opus/Sonnet/Haiku)
- Validacion automatica de estructura y gobernanza (SK-004)
- Flujo de revision con firma digital y auditoria completa
- Prompt builder que genera prompts estructurados trazables
- Dashboard de metricas convergentes (calidad, costos, trazabilidad, DLP)
- Catalogo de skills con metricas y alertas de retrabajo
- Sincronizacion automatica con Azure DevOps (Feature/User Story/Task)
- DLP bidireccional: pre-prompt y post-response

### Que NO hace
- No ejecuta el codigo generado (solo genera prompts)
- No reemplaza el CI/CD del cliente (complementa con quality gates)
- No es un IDE ni editor de codigo
- No gestiona repositorios Git directamente (referencia URL)
- No implementa DAST (requiere ambiente desplegado)

---

## Industrias aplicables

| Industria | Aplica | Notas de adaptacion |
|-----------|--------|-------------------|
| Financiera | Si | Enfasis en clasificacion de datos (PII financiero), compliance regulatorio, ADRs de seguridad |
| Salud | Si | Datos de salud como PII critico, compliance HIPAA/Habeas Data, masking estricto |
| Retail / E-commerce | Si | Menor enfasis en PII, mas en velocidad de iteracion y metricas de entrega |
| Logistica | Si | Datos de tracking como Interno, integraciones con sistemas legacy |
| Gobierno | Si | Compliance estricto, clasificacion de datos segun normativa local, audit trail completo |
| SaaS / Startups | Si | Configuracion ligera, enfasis en velocidad con gobernanza minima viable |
| **Universal** | **Si** | El acelerador aplica a cualquier equipo que genere codigo con IA y necesite gobernanza |

---

## Componentes reutilizables (base)

Estos componentes son **iguales** para cualquier cliente/dominio:

| Componente | Descripcion |
|-----------|------------|
| Spec Management Service | CRUD de specs L1/L2/L3 con versionado semver, jerarquia parent-child, almacenamiento JSONB |
| Governance Service | Gestion de proyectos, ADRs, quality gates. Gate 1 automatizado |
| Agent Orchestration | Generacion asincrona con LLM, seleccion de modelo, DLP bidireccional |
| Review Service | Flujo DRAFT → IN_REVIEW → APPROVED con validacion SK-004 |
| Prompt Builder | Construccion local de prompts estructurados con DLP automatico |
| Metrics Service | Dashboard con cache Redis, arbol de trazabilidad, exportacion |
| Skill Catalog | Registro, versionado, deprecacion y metricas de skills |
| Auth + RBAC | OAuth 2.0 con 5 roles (ARCHITECT, LEAD, SENIOR_DEV, QA, PO) |
| DLP Filter | Filtro bidireccional regex-based configurable por proyecto |
| Audit Log | Registro inmutable de acciones criticas con correlationId |

---

## Puntos de personalizacion

Estos componentes **cambian** segun el cliente/dominio:

| Punto | Que se personaliza | Ejemplo |
|-------|-------------------|---------|
| ADRs del proyecto | Cada cliente define sus propios ADRs segun su arquitectura | Cliente financiero: ADR de cifrado AES-256 obligatorio |
| Quality gates | Validaciones especificas del cliente | Cliente salud: validacion HIPAA como CRITICAL |
| Clasificacion de datos | Niveles y tratamiento segun normativa del cliente | Colombia: Ley 1581. EU: GDPR. US: HIPAA |
| Templates de spec | Estructura de secciones por nivel (L1/L2/L3) | Cliente con microservicios: seccion de eventos de dominio obligatoria |
| Patrones DLP | Regex y patrones segun tipo de datos del cliente | Financiero: patrones de cuenta bancaria, NIT |
| Integracion DevOps | Azure DevOps, Jira, Linear, GitHub Issues | Cada cliente usa su tool de backlog |
| Proveedor LLM | Claude, OpenAI, Azure OpenAI, modelos on-premise | Cliente con restriccion de datos: modelo on-premise |
| Proveedor OAuth | GitHub, Azure AD, Okta, Auth0 | Cada cliente tiene su identity provider |
| Metricas custom | KPIs especificos del cliente | Compliance rate, security score |
| Skills custom | Skills especificos del dominio del cliente | Generacion de reportes regulatorios, generacion de IaC |

---

## Arquitectura tecnica

```
┌────────────────��──────────��─────────────────────────────┐
│                    FRONTEND (Angular 21)                  │
│  Projects | Specs | Reviews | Prompts | Metrics | Skills │
└──────────────────────────┬──────────────────────────────┘
                           │
┌──────────────────────────▼──────────────────────────────┐
│                    BACKEND (.NET 10)                      │
│                                                          │
│  ┌───────────���─────────────────────────────────────���──┐  │
│  │              APPLICATION LAYER                      │  │
│  │  GovernanceSvc | SpecMgmtSvc | ReviewSvc           │  │
│  │  PromptBuilder | MetricsSvc  | SkillCatalogSvc     │  │
│  │  DevOpsSyncSvc                                      │  │
│  └────────────────────────────────────────────────────┘  │
│  ┌───────────────────────────────────────────��────────┐  │
│  │              DOMAIN LAYER (Ports)                   │  │
│  │  10 entidades | 14 puertos | Value Objects         │  │
│  └────────────────────────────────────────────────────┘  │
│  ┌─────────────────────────────────────��──────────────┐  │
│  │           INFRASTRUCTURE (Adapters)                 │  │
│  │  PostgreSQL | Redis | Service Bus | Claude API     │  │
│  │  Azure DevOps | OAuth | DLP Filter | Seq           │  │
│  └────────────────────────────────────────────────────┘  │
└────────────────────────��─────────────────────────────────┘
```

### Stack tecnologico

| Componente | Tecnologia |
|-----------|-----------|
| Frontend | Angular 21 + TypeScript + Angular Material + Tailwind CSS |
| Editor | Monaco Editor (ngx-monaco-editor) |
| Backend | .NET 10 + C# + Minimal APIs |
| IA | Semantic Kernel + Claude API (Anthropic) |
| Base de datos | PostgreSQL 16 (JSONB) |
| Cache | Redis 7 |
| Cola | Azure Service Bus (o RabbitMQ) |
| Auth | OAuth 2.0 (configurable) |
| Observabilidad | OpenTelemetry + Seq |
| Deployment | Docker Compose (MVP) → Kubernetes (scale) |

---

## Integraciones

| Sistema externo | Tipo de integracion | Proposito |
|----------------|-------------------|----------|
| Claude API (Anthropic) | REST API | Generacion de specs L1/L2/L3 y ADRs asistidos |
| Azure DevOps | REST API v7.1 | Sincronizacion automatica spec → work items |
| GitHub / Azure AD | OAuth 2.0 | Autenticacion de usuarios |
| Azure Service Bus | AMQP | Cola de jobs para generacion asincrona |
| Seq | HTTP (Serilog sink) | Centralizacion de logs estructurados |

---

## Skills que compone

| Skill ID | Nombre | Rol en este acelerador |
|----------|--------|----------------------|
| SK-001 | Generacion de caso de uso | Genera implementacion desde spec L2 aprobada |
| SK-002 | Generacion de tests GWT | Genera tests antes de implementar (TDD) |
| SK-003 | Generacion de contrato API | Genera OpenAPI desde spec L2 |
| SK-004 | Revision automatizada de spec | Valida estructura automaticamente (Gate 1) |
| SK-005 | Pipeline de validacion PR | CI/CD con SAST + SCA + DLP (Gate 2) |
| SK-006 | Data Masking | Genera filtros de masking para campos PII |
| SK-007 | DLP Guard LLM | Middleware DLP bidireccional para integracion LLM |

---

## Metricas de exito

| Metrica | Como se mide | Objetivo |
|---------|-------------|---------|
| Tiempo de creacion de spec L1 | Desde input hasta DRAFT guardado | < 30 min (antes: 2-4 hrs manual) |
| Cobertura de clasificacion de datos | % de L1 con clasificacion completa | 100% |
| Tiempo de ciclo de aprobacion | Desde DRAFT hasta APPROVED | < 1 dia (antes: 2-5 dias) |
| % reutilizacion de componentes base | Componentes sin modificar / total | > 70% |
| Tiempo de setup por cliente | Desde clon hasta portal operativo | < 1 dia |
| Retrabajo en specs generadas | % de specs que requieren > 2 ciclos de revision | < 15% |
| PII detectada en pipeline | Hallazgos de PII en PRs | 0 por sprint |

---

## Plan de validacion

1. **Piloto interno**: Proyecto "Account Planning" de Tech&Solve — ya tiene specs L1 y L2 como referencia
2. **Piloto con cliente**: Primer cliente que adopte framework AI-Driven Engineering (enfoque: dominio acotado, 1 equipo)
3. **Feedback**: Encuesta a usuarios del portal cada sprint + metricas de uso automaticas (dashboard)
4. **Iteracion**: Ajustar templates, patrones DLP, y skills basandose en metricas de retrabajo y tiempos de revision

---

## Estimacion de esfuerzo para replicar

| Fase | Esfuerzo | Notas |
|------|---------|-------|
| Setup infra (Docker + DB + Service Bus) | 1 dia | Con docker-compose.yml incluido |
| Personalizacion (ADRs + gates + templates) | 2-3 dias | Segun complejidad del dominio |
| Configuracion OAuth + roles | 0.5 dia | Depende del identity provider |
| Integracion Azure DevOps | 0.5 dia | Configurar desde UI del portal |
| Capacitacion del equipo | 1 dia | Workshop de framework + portal |
| **Total estimado** | **5-6 dias** | Para tener el portal operativo con un proyecto |

---

## Historial de cambios

| Version | Fecha | Autor | Descripcion |
|---------|-------|-------|------------|
| 1.0.0 | 2026-04-09 | Equipo de Arquitectura | Version inicial del acelerador |
