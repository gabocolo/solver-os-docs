# Arquitectura de Solucion: Account Planning con IA

## 1. Vision general

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                         ACCOUNT PLANNING INTELIGENTE                            │
│                                                                                 │
│  ┌─────────────┐   ┌──────────────┐   ┌──────────────┐   ┌──────────────────┐  │
│  │  Frontend    │   │   Backend    │   │  AI Engine   │   │   Data Layer     │  │
│  │  (Angular)   │──▶│  (.NET 8)    │──▶│  (Python +   │   │  (SQL Server +   │  │
│  │             │   │              │   │   Claude API) │   │   Redis + Blob)  │  │
│  └─────────────┘   └──────────────┘   └──────────────┘   └──────────────────┘  │
│         │                  │                  │                     │            │
│         └──────────────────┴──────────────────┴─────────────────────┘            │
│                                     │                                           │
│                          ┌──────────┴──────────┐                                │
│                          │   Event Bus (Redis   │                                │
│                          │   Streams / Azure    │                                │
│                          │   Service Bus)       │                                │
│                          └─────────────────────┘                                │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Principios de arquitectura

| Principio | Aplicacion |
|-----------|-----------|
| **Hexagonal (ADR-001)** | Backend con puertos (interfaces C#) y adaptadores. Dominio aislado de infraestructura |
| **API-First** | Contrato OpenAPI definido desde la spec L2 antes de implementar |
| **Event-Driven** | Eventos para operaciones asincronas (scoring, notificaciones) |
| **AI as a Service** | Motor de IA desacoplado del backend via puerto, intercambiable |
| **Progressive Enhancement** | MVP funcional sin IA → IA como mejora, no como dependencia critica |

---

## 3. Stack tecnologico

### Frontend

| Tecnologia | Justificacion |
|-----------|---------------|
| **Angular 18+** | Framework enterprise probado, tipado estricto, DI nativa, estructura predecible para equipos con rotacion. Stack dominante en clientes de Tech&Solve |
| **TypeScript** | Tipado fuerte alineado con contratos de la spec L2 |
| **Angular Material + Tailwind CSS** | Componentes enterprise + utilidades de diseño rapido |
| **NgRx (Signals)** | State management reactivo, predecible, alineado con Angular moderno |
| **ngx-charts** | Graficas de health score, pipelines, tendencias de scoring |
| **Angular Reactive Forms + Zod** | Formularios complejos (wizard de plan) con validacion tipada |

### Backend

| Tecnologia | Justificacion |
|-----------|---------------|
| **.NET 8 (ASP.NET Core Web API)** | Performance superior (Kestrel), interfaces como ciudadano de primera clase para hexagonal, DI container nativo, stack dominante en clientes de Tech&Solve |
| **C# 12** | Pattern matching, records para value objects, interfaces para puertos |
| **Entity Framework Core 8** | ORM maduro, migraciones versionadas, LINQ type-safe |
| **MediatR** | CQRS + mediator para casos de uso desacoplados |
| **Hangfire** | Jobs de auto-scoring periodico, procesamiento asincrono con dashboard |
| **ASP.NET Core Identity + JWT** | Autenticacion, roles (Account Manager, Director, Admin) |
| **FluentValidation** | Validacion de inputs segun spec L2, expresiva y testeable |

### Motor de IA

| Tecnologia | Justificacion |
|-----------|---------------|
| **Python (FastAPI)** | Ecosistema de IA maduro, integracion nativa con SDKs de LLMs. No hay equivalente en .NET para este nivel de madurez |
| **Claude API (Anthropic SDK)** | Modelo de frontera, contexto largo para analizar multiples procesos |
| **LangChain (opcional)** | Chains para analisis multi-paso, structured output |
| **Pydantic** | Validacion y schemas de entrada/salida del motor |

**Modelo recomendado por tarea:**

| Tarea | Modelo | Razon |
|-------|--------|-------|
| Analisis de procesos + oportunidades | **Claude Sonnet** | Balance costo/calidad para analisis acotado |
| Generacion de rationale y recomendaciones | **Claude Sonnet** | Buena calidad narrativa |
| Matching con aceleradores | **Claude Haiku** | Tarea de clasificacion, rapida y barata |
| Decisiones de arquitectura del sistema | **Claude Opus** | Solo para disenar, no en runtime |

### Data Layer

| Tecnologia | Justificacion |
|-----------|---------------|
| **SQL Server 2022 / Azure SQL** | Relacional enterprise, excelente integracion con EF Core, familiar para equipos .NET. Alternativa: PostgreSQL si el cliente lo prefiere |
| **Redis** | Cache de catalogo de aceleradores, session store, pub/sub para eventos |
| **Azure Blob Storage / Amazon S3** | Almacenamiento de reportes generados, exports, attachments |

### Infraestructura

| Tecnologia | Justificacion |
|-----------|---------------|
| **Docker + Docker Compose** | Desarrollo local reproducible |
| **Azure (App Service / Container Apps)** | Nativo para .NET, auto-scaling, integrado con Azure SQL y Redis. Alternativa: AWS ECS si el cliente usa AWS |
| **Azure DevOps o GitHub Actions** | CI/CD con quality gates (SAST, SCA, tests) |
| **Terraform** | IaC para infraestructura (alineado con vision de Jonathan Loste) |

---

## 4. Arquitectura de componentes

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                            FRONTEND (Angular 18)                            │
│                                                                             │
│  ┌─────────────┐  ┌──────────────┐  ┌────────────┐  ┌──────────────────┐   │
│  │  Dashboard   │  │  Account     │  │ Opportunity │  │  Reports &       │   │
│  │  Module      │  │  Plan Module │  │ Module      │  │  Analytics Mod.  │   │
│  └──────┬──────┘  └──────┬───────┘  └─────┬──────┘  └────────┬─────────┘   │
│         └────────────────┴────────────────┴───────────────────┘             │
│                                    │ HTTP (HttpClient)                       │
└────────────────────────────────────┼────────────────────────────────────────┘
                                     │
┌────────────────────────────────────┼────────────────────────────────────────┐
│                         BACKEND (.NET 8 Web API)                            │
│                                    │                                        │
│  ┌──────────────────── API Layer ──┴──────────────────────────────────┐     │
│  │  PlanController  │  AccountController  │  OpportunityController    │     │
│  │  StakeholderController  │  ReportController  │  HealthController   │     │
│  │                                                                    │     │
│  │  DTOs (Records) ←→ FluentValidation ←→ AutoMapper                  │     │
│  └────────────────────────────┬───────────────────────────────────────┘     │
│                                │                                            │
│  ┌──────────────── Application Layer (MediatR Handlers) ─────────────┐     │
│  │                                                                    │     │
│  │  CreateAccountPlanHandler     IdentifyOpportunitiesHandler         │     │
│  │  UpdatePlanHandler            RecalculateScoringHandler            │     │
│  │  ManageStakeholdersHandler    GenerateReportHandler                │     │
│  │  LogInteractionHandler        MatchAcceleratorsHandler             │     │
│  │                                                                    │     │
│  └────────────────────────────┬───────────────────────────────────────┘     │
│                                │                                            │
│  ┌──────────────── Domain Layer ─────────────────────────────────────┐     │
│  │                                                                    │     │
│  │  Entities:     Account, Plan, Stakeholder, Opportunity, ActionItem │     │
│  │  Value Objs:   HealthScore (record), OpportunityScore, ROIEstimate │     │
│  │  Interfaces:   IAccountRepository, IPlanRepository,                │     │
│  │                IAIEngine, IAcceleratorCatalog,                     │     │
│  │                IHealthCalculator, IEventPublisher,                 │     │
│  │                IScoringScheduler                                   │     │
│  │  Domain Svcs:  ScoringPolicy, StakeholderCoverageAnalyzer          │     │
│  │                                                                    │     │
│  └────────────────────────────┬───────────────────────────────────────┘     │
│                                │                                            │
│  ┌──────────────── Infrastructure Layer ─────────────────────────────┐     │
│  │                                                                    │     │
│  │  Adapters:                                                         │     │
│  │    EfAccountRepository     (→ SQL Server via EF Core)              │     │
│  │    EfPlanRepository        (→ SQL Server via EF Core)              │     │
│  │    RedisEventPublisher     (→ Redis Streams)                       │     │
│  │    AIEngineHttpClient      (→ Python AI Service via HttpClient)    │     │
│  │    CachedAcceleratorCatalog(→ SQL Server + Redis cache)            │     │
│  │    HangfireScoringScheduler(→ Hangfire + Redis)                    │     │
│  │                                                                    │     │
│  │  Config:                                                           │     │
│  │    AppDbContext (EF Core)                                          │     │
│  │    DependencyInjection.cs  (registro de puertos → adaptadores)    │     │
│  └────────────────────────────────────────────────────────────────────┘     │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
                          │                    │
              ┌───────────┘                    └───────────┐
              ▼                                            ▼
┌──────────────────────┐                    ┌──────────────────────────┐
│  AI ENGINE (FastAPI)  │                    │     DATA STORES          │
│                       │                    │                          │
│  /analyze-processes   │                    │  SQL Server / Azure SQL  │
│  /match-accelerators  │                    │    ├── Accounts          │
│  /calculate-roi       │                    │    ├── Plans             │
│  /rescore             │                    │    ├── Stakeholders      │
│                       │                    │    ├── Opportunities     │
│  Claude API ─────┐    │                    │    ├── Accelerators      │
│  (Sonnet/Haiku)  │    │                    │    ├── Interactions      │
│                  ▼    │                    │    └── ScoreHistory      │
│  Prompt Templates     │                    │                          │
│  Structured Output    │                    │  Redis                   │
│  Response Parsing     │                    │    ├── cache             │
│                       │                    │    ├── sessions          │
└───────────────────────┘                    │    ├── Hangfire queues   │
                                             │    └── events (Streams) │
                                             │                          │
                                             │  Azure Blob / S3         │
                                             │    └── reports, exports  │
                                             └──────────────────────────┘
```

---

## 5. Decisiones de arquitectura clave

### D1: Backend .NET y AI Engine Python — separados

```
Backend (.NET 8)  ───HTTP (HttpClient)───▶  AI Engine (Python/FastAPI)
```

**Por que separados:** El ecosistema de IA en Python es superior (Anthropic SDK nativo, LangChain, numpy). .NET no tiene equivalente maduro para orquestar LLMs con structured output. Separarlos permite:
- Escalar el AI Engine independientemente
- Cambiar el modelo de IA sin tocar el backend
- Si la IA cae, el plan se crea sin sugerencias (progressive enhancement)
- Los equipos .NET no necesitan aprender Python y viceversa

**Por que .NET para el backend:** Interfaces nativas para hexagonal, DI container superior, EF Core maduro, rendimiento de Kestrel, y es el stack que los equipos de Tech&Solve ya dominan.

### D2: MediatR para casos de uso (CQRS light)

```
Controller → MediatR.Send(CreateAccountPlanCommand) → CreateAccountPlanHandler
```

**Por que:** Cada caso de uso de la spec L2 se mapea 1:1 a un MediatR handler. Esto mantiene los controllers delgados y los casos de uso desacoplados. Pipeline behaviors de MediatR permiten agregar logging, validacion y transacciones de forma transversal.

### D3: Event-driven para operaciones asincronas

```
CreateAccountPlanHandler ──emit──▶ AccountPlanCreated ──consume──▶ NotificationService
RecalculateScoringHandler ──emit──▶ OpportunityScoreChanged ──consume──▶ AlertService
```

**Por que:** El auto-scoring es periodico y pesado. Los eventos desacoplan la creacion del plan de las notificaciones y reportes.

### D4: Progressive Enhancement con IA

```
Plan SIN IA:  Perfil + Stakeholders + Procesos + Oportunidades manuales = Plan basico funcional
Plan CON IA:  Todo lo anterior + Oportunidades sugeridas + Matching + ROI + Scoring = Plan inteligente
```

**Por que:** Si Claude API esta caido, el sistema sigue funcionando. La IA mejora, no bloquea.

### D5: Cache del catalogo de aceleradores

```
AcceleratorCatalog (SQL Server) ──cache──▶ Redis (TTL: 1h) ──read──▶ Backend
```

**Por que:** El catalogo cambia poco pero se consulta mucho. IMemoryCache + Redis como cache distribuido.

---

## 6. Ventajas de .NET para esta arquitectura

### Hexagonal nativo con interfaces C#

```csharp
// Puerto (Domain Layer) — interface nativa de C#
public interface IAIEngine
{
    Task<IReadOnlyList<OpportunitySuggestion>> AnalyzeProcessesAsync(
        AnalysisRequest request, CancellationToken ct);
}

// Adaptador (Infrastructure Layer) — implementa el puerto
public class AIEngineHttpClient : IAIEngine
{
    private readonly HttpClient _httpClient;

    public async Task<IReadOnlyList<OpportunitySuggestion>> AnalyzeProcessesAsync(
        AnalysisRequest request, CancellationToken ct)
    {
        var response = await _httpClient.PostAsJsonAsync("/analyze-processes", request, ct);
        return await response.Content.ReadFromJsonAsync<List<OpportunitySuggestion>>(ct);
    }
}

// Registro en DI (Program.cs)
builder.Services.AddScoped<IAIEngine, AIEngineHttpClient>();
```

### Value Objects con records

```csharp
// Value objects inmutables — mapeo directo de la spec
public record HealthScore(
    int Overall,
    int Satisfaction,
    int Engagement,
    int GrowthPotential,
    int ServiceHealth);

public record ROIEstimate(
    decimal CurrentCostPerMonth,
    decimal ProjectedCostPerMonth,
    decimal MonthlySavings,
    decimal ImplementationCost,
    int PaybackMonths);
```

### Caso de uso con MediatR

```csharp
// Command desde la spec L2
public record CreateAccountPlanCommand(
    Guid AccountId,
    PlanPeriod Period,
    Guid OwnerId,
    AccountProfileDto AccountProfile,
    List<StakeholderDto> Stakeholders,
    List<ClientProcessDto> ClientProcesses,
    List<ManualOpportunityDto>? ManualOpportunities,
    List<string> Objectives,
    Guid IdempotencyKey) : IRequest<AccountPlanResponse>;

// Handler — caso de uso (Application Layer)
public class CreateAccountPlanHandler
    : IRequestHandler<CreateAccountPlanCommand, AccountPlanResponse>
{
    private readonly IAccountRepository _accounts;
    private readonly IPlanRepository _plans;
    private readonly IAIEngine _aiEngine;
    private readonly IAcceleratorCatalog _catalog;
    private readonly IHealthCalculator _health;
    private readonly IEventPublisher _events;

    // Constructor con DI de todos los puertos...

    public async Task<AccountPlanResponse> Handle(
        CreateAccountPlanCommand cmd, CancellationToken ct)
    {
        // 1. Validar cuenta existe
        // 2. Verificar idempotencia
        // 3. Analizar stakeholder coverage
        // 4. Calcular health score
        // 5. Llamar AI Engine (con fallback si no disponible)
        // 6. Merge oportunidades manual + IA
        // 7. Generar action plan
        // 8. Persistir (transaccion atomica)
        // 9. Emitir evento AccountPlanCreated
    }
}
```

---

## 7. Seguridad

| Capa | Medida |
|------|--------|
| **Autenticacion** | ASP.NET Core Identity + JWT + refresh tokens, Azure AD / OAuth2 para SSO corporativo |
| **Autorizacion** | RBAC con policies: Admin, Director, AccountManager, Viewer |
| **API** | Rate limiting (AspNetCoreRateLimit), CORS, security headers, FluentValidation |
| **Datos** | Transparent Data Encryption (SQL Server), TLS en transito, PII anonimizada en logs |
| **IA** | Prompts sin datos sensibles del cliente (solo procesos y descripciones), no enviar PII al LLM |
| **Infra** | VNet/VPC, NSG/security groups, Azure Key Vault para secrets, no credenciales en codigo |
| **Pipeline** | SAST + SCA en CI/CD (Gate 2), dependencias auditadas con `dotnet list package --vulnerable` |

### Roles y permisos

| Rol | Puede hacer |
|-----|------------|
| **Admin** | Todo + gestionar usuarios + configurar sistema |
| **Director** | Ver todos los planes + dashboards + aprobar oportunidades > $50K |
| **Account Manager** | CRUD de sus planes + analisis IA + registrar interacciones |
| **Viewer** | Solo lectura de planes asignados |

---

## 8. Observabilidad

```
┌─────────────┐     ┌──────────────┐     ┌──────────────┐
│   Backend    │────▶│  OpenTelemetry│────▶│  Grafana     │
│   (.NET)     │     │  Collector    │     │  (dashboards)│
│   AI Engine  │     └──────┬───────┘     └──────────────┘
│   (Python)   │            │
│   Frontend   │     ┌──────┴───────┐
│   (Angular)  │     │              │
└─────────────┘┌─────▼────┐  ┌─────▼─────┐
               │  Loki     │  │ Prometheus │
               │  (logs)   │  │ (metricas) │
               └──────────┘  └───────────┘
```

.NET tiene soporte nativo para OpenTelemetry:
```csharp
builder.Services.AddOpenTelemetry()
    .WithTracing(tracing => tracing
        .AddAspNetCoreInstrumentation()
        .AddHttpClientInstrumentation()
        .AddEntityFrameworkCoreInstrumentation())
    .WithMetrics(metrics => metrics
        .AddAspNetCoreInstrumentation()
        .AddHttpClientInstrumentation());
```

| Pilar | Herramienta | Que captura |
|-------|------------|------------|
| **Logs** | Serilog + Loki | Logs estructurados JSON, correlation IDs |
| **Trazas** | OpenTelemetry + Jaeger/Tempo | Request → .NET → AI Engine → DB |
| **Metricas** | Prometheus + Grafana | Latencia, throughput, error rate, AI response time |
| **Alertas** | Grafana Alerting / Azure Monitor | Error rate > 5%, AI timeout > 10%, health score drops |

Alternativa Azure nativa: **Application Insights** (integrado con .NET sin configuracion adicional).

---

## 9. Ambientes

| Ambiente | Proposito | Infra |
|----------|----------|-------|
| **Local** | Desarrollo | Docker Compose (SQL Server container, Redis, AI Engine mock opcional) |
| **Dev** | Integracion continua | Azure App Service / Container Apps, Azure SQL, Redis |
| **Staging** | Pre-produccion, demos | Replica de prod, datos anonimizados |
| **Prod** | Produccion | HA, backups, monitoring completo |

---

*Documento de arquitectura v2.0 — Account Planning Inteligente — Tech&Solve — Stack: Angular + .NET 8 + Python (AI)*
