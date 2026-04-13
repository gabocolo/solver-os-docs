# Deployment e infraestructura

## Arquitectura de deployment

```
                         ┌─────────────────────────────────────┐
                         │           INTERNET                   │
                         └──────────────┬──────────────────────┘
                                        │
                                        ▼
                         ┌──────────────────────────────────────┐
                         │       LOAD BALANCER / GATEWAY        │
                         │  (Azure Front Door / Application GW) │
                         │      SSL termination + WAF           │
                         └──────┬───────────────┬───────────────┘
                                │               │
                    ┌───────────▼──┐    ┌───────▼────────────┐
                    │  FRONTEND    │    │     BACKEND         │
                    │  (Angular)   │    │     (.NET 8 API)    │
                    │              │    │                     │
                    │  Azure Static│    │  Azure App Service  │
                    │  Web App /   │    │  o Container Apps   │
                    │  Nginx in    │    │  x2-4 instances     │
                    │  container   │    │  (auto-scaling)     │
                    │              │    │                     │
                    │  Port: 80    │    │  Port: 5000         │
                    └──────────────┘    └──────┬──────────────┘
                                               │
                              ┌────────────────┼────────────────┐
                              │                │                │
                    ┌─────────▼──┐   ┌────────▼───────┐  ┌────▼─────────┐
                    │ AI ENGINE  │   │  SQL Server     │  │    Redis     │
                    │ (FastAPI)  │   │  (Azure SQL)    │  │  (Azure     │
                    │            │   │                 │  │   Cache for  │
                    │ Container  │   │  Primary +      │  │   Redis)     │
                    │ App x1-2   │   │  Read replica   │  │              │
                    │            │   │                 │  │  Queues +    │
                    │ Port: 8000 │   │  Port: 1433     │  │  Cache +     │
                    └─────┬──────┘   └─────────────────┘  │  Hangfire    │
                          │                                │  Port: 6379  │
                          ▼                                └──────────────┘
                    ┌──────────────┐
                    │  Claude API  │
                    │  (Anthropic) │
                    │  External    │
                    └──────────────┘
```

---

## Docker Compose (desarrollo local)

```yaml
# docker-compose.yml
version: '3.8'

services:
  # ─── Frontend (Angular) ─────────────────────
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    ports:
      - "4200:80"
    environment:
      - API_URL=http://localhost:5000
    depends_on:
      - backend

  # ─── Backend (.NET 8 API) ──────────────────
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    ports:
      - "5000:5000"
    environment:
      - ConnectionStrings__DefaultConnection=Server=db;Database=AccountPlanning;User=sa;Password=YourStr0ngP@ssword;TrustServerCertificate=True
      - ConnectionStrings__Redis=redis:6379
      - AIEngine__BaseUrl=http://ai-engine:8000
      - Jwt__Secret=${JWT_SECRET}
      - ASPNETCORE_ENVIRONMENT=Development
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_started

  # ─── AI Engine (Python FastAPI) ─────────────
  ai-engine:
    build:
      context: ./ai-engine
      dockerfile: Dockerfile
    ports:
      - "8000:8000"
    environment:
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
      - DEFAULT_MODEL=claude-sonnet-4-6
      - MATCHING_MODEL=claude-haiku-4-5-20251001
      - REDIS_URL=redis://redis:6379
    depends_on:
      db:
        condition: service_healthy

  # ─── SQL Server ────────────────────────────
  db:
    image: mcr.microsoft.com/mssql/server:2022-latest
    ports:
      - "1433:1433"
    environment:
      - ACCEPT_EULA=Y
      - MSSQL_SA_PASSWORD=YourStr0ngP@ssword
    volumes:
      - sqldata:/var/opt/mssql
    healthcheck:
      test: /opt/mssql-tools18/bin/sqlcmd -S localhost -U sa -P "YourStr0ngP@ssword" -C -Q "SELECT 1" || exit 1
      interval: 10s
      timeout: 5s
      retries: 5

  # ─── Redis ─────────────────────────────────
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redisdata:/data

volumes:
  sqldata:
  redisdata:
```

---

## Dockerfiles

### Backend (.NET 8)
```dockerfile
# backend/Dockerfile
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY *.sln .
COPY src/**/*.csproj ./src/
RUN dotnet restore
COPY . .
RUN dotnet publish src/AccountPlanning.Api/AccountPlanning.Api.csproj \
    -c Release -o /app/publish

FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS runtime
WORKDIR /app
COPY --from=build /app/publish .
EXPOSE 5000
ENV ASPNETCORE_URLS=http://+:5000
ENTRYPOINT ["dotnet", "AccountPlanning.Api.dll"]
```

### Frontend (Angular)
```dockerfile
# frontend/Dockerfile
FROM node:20-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build -- --configuration production

FROM nginx:alpine AS runtime
COPY --from=build /app/dist/account-planning/browser /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
```

---

## Estructura de repositorio

```
account-planning/
├── docker-compose.yml
├── .github/
│   └── workflows/
│       ├── ci.yml                    ← Tests + SAST + SCA (Gate 2)
│       └── deploy.yml                ← Deploy a staging/prod
│
├── frontend/                         ← Angular App
│   ├── Dockerfile
│   ├── nginx.conf
│   ├── package.json
│   ├── angular.json
│   ├── src/
│   │   ├── app/
│   │   │   ├── core/                 ← Services, guards, interceptors
│   │   │   │   ├── services/
│   │   │   │   │   ├── plan.service.ts
│   │   │   │   │   ├── account.service.ts
│   │   │   │   │   ├── opportunity.service.ts
│   │   │   │   │   └── auth.service.ts
│   │   │   │   ├── interceptors/
│   │   │   │   │   └── auth.interceptor.ts
│   │   │   │   └── guards/
│   │   │   │       └── role.guard.ts
│   │   │   ├── features/             ← Feature modules
│   │   │   │   ├── dashboard/
│   │   │   │   │   ├── dashboard.component.ts
│   │   │   │   │   └── dashboard.routes.ts
│   │   │   │   ├── accounts/
│   │   │   │   │   ├── account-list/
│   │   │   │   │   └── account-detail/
│   │   │   │   ├── plans/
│   │   │   │   │   ├── plan-wizard/       ← Stepper de creacion
│   │   │   │   │   │   ├── step-profile/
│   │   │   │   │   │   ├── step-stakeholders/
│   │   │   │   │   │   ├── step-processes/
│   │   │   │   │   │   └── step-review/
│   │   │   │   │   └── plan-detail/
│   │   │   │   ├── opportunities/
│   │   │   │   │   ├── opportunity-explorer/
│   │   │   │   │   └── pipeline-board/
│   │   │   │   └── reports/
│   │   │   ├── shared/               ← Componentes compartidos
│   │   │   │   ├── health-gauge/
│   │   │   │   ├── stakeholder-map/
│   │   │   │   ├── opportunity-card/
│   │   │   │   ├── score-trend-chart/
│   │   │   │   └── pipeline-summary/
│   │   │   ├── models/              ← Interfaces TypeScript (from spec)
│   │   │   └── app.routes.ts
│   │   └── environments/
│   └── e2e/
│
├── backend/                          ← .NET 8 Solution
│   ├── Dockerfile
│   ├── AccountPlanning.sln
│   ├── src/
│   │   ├── AccountPlanning.Domain/          ← Capa de dominio (Class Library)
│   │   │   ├── Entities/
│   │   │   │   ├── Account.cs
│   │   │   │   ├── AccountPlan.cs
│   │   │   │   ├── Opportunity.cs
│   │   │   │   ├── Stakeholder.cs
│   │   │   │   ├── ActionItem.cs
│   │   │   │   └── Interaction.cs
│   │   │   ├── ValueObjects/
│   │   │   │   ├── HealthScore.cs          ← record struct
│   │   │   │   ├── OpportunityScore.cs
│   │   │   │   └── ROIEstimate.cs
│   │   │   ├── Enums/
│   │   │   │   └── Enums.cs
│   │   │   ├── Interfaces/                 ← PUERTOS (core del hexagonal)
│   │   │   │   ├── IAccountRepository.cs
│   │   │   │   ├── IPlanRepository.cs
│   │   │   │   ├── IAIEngine.cs
│   │   │   │   ├── IAcceleratorCatalog.cs
│   │   │   │   ├── IHealthCalculator.cs
│   │   │   │   ├── IScoringScheduler.cs
│   │   │   │   └── IEventPublisher.cs
│   │   │   └── Services/
│   │   │       ├── ScoringPolicy.cs
│   │   │       └── StakeholderCoverageAnalyzer.cs
│   │   │
│   │   ├── AccountPlanning.Application/     ← Capa de aplicacion (Class Library)
│   │   │   ├── Commands/
│   │   │   │   ├── CreateAccountPlan/
│   │   │   │   │   ├── CreateAccountPlanCommand.cs
│   │   │   │   │   ├── CreateAccountPlanHandler.cs
│   │   │   │   │   └── CreateAccountPlanValidator.cs
│   │   │   │   ├── LogInteraction/
│   │   │   │   └── RecalculateScoring/
│   │   │   ├── Queries/
│   │   │   │   ├── GetPlanById/
│   │   │   │   ├── GetAccountPlans/
│   │   │   │   └── GetPipelineSummary/
│   │   │   ├── DTOs/
│   │   │   └── Mappings/
│   │   │       └── MappingProfile.cs        ← AutoMapper
│   │   │
│   │   ├── AccountPlanning.Infrastructure/  ← Capa de infraestructura (Class Library)
│   │   │   ├── Persistence/
│   │   │   │   ├── AppDbContext.cs           ← EF Core DbContext
│   │   │   │   ├── Configurations/          ← Fluent API configs por entidad
│   │   │   │   │   ├── AccountConfiguration.cs
│   │   │   │   │   ├── PlanConfiguration.cs
│   │   │   │   │   └── OpportunityConfiguration.cs
│   │   │   │   ├── Migrations/
│   │   │   │   └── Repositories/
│   │   │   │       ├── EfAccountRepository.cs
│   │   │   │       └── EfPlanRepository.cs
│   │   │   ├── ExternalServices/
│   │   │   │   ├── AIEngineHttpClient.cs    ← Adaptador → Python AI Engine
│   │   │   │   └── CachedAcceleratorCatalog.cs
│   │   │   ├── BackgroundJobs/
│   │   │   │   └── HangfireScoringJob.cs
│   │   │   ├── Events/
│   │   │   │   └── RedisEventPublisher.cs
│   │   │   └── DependencyInjection.cs       ← Registro de puertos → adaptadores
│   │   │
│   │   └── AccountPlanning.Api/             ← ASP.NET Core Web API (Startup)
│   │       ├── Controllers/
│   │       │   ├── PlanController.cs
│   │       │   ├── AccountController.cs
│   │       │   ├── OpportunityController.cs
│   │       │   └── InteractionController.cs
│   │       ├── Middleware/
│   │       │   └── ExceptionHandlingMiddleware.cs
│   │       ├── Program.cs
│   │       └── appsettings.json
│   │
│   └── tests/
│       ├── AccountPlanning.UnitTests/       ← Tests unitarios (xUnit)
│       │   └── UseCases/
│       │       ├── CreateAccountPlanTests.cs
│       │       └── IdentifyOpportunitiesTests.cs
│       └── AccountPlanning.IntegrationTests/
│           └── Controllers/
│
├── ai-engine/                        ← Python FastAPI (sin cambios)
│   ├── Dockerfile
│   ├── requirements.txt
│   ├── src/
│   │   ├── main.py
│   │   ├── routes/
│   │   ├── services/
│   │   ├── prompts/
│   │   ├── schemas/
│   │   └── config.py
│   └── tests/
│
├── infrastructure/                   ← IaC (Terraform)
│   ├── modules/
│   │   ├── networking/               ← VNet, NSG, subnets
│   │   ├── database/                 ← Azure SQL + Redis
│   │   ├── compute/                  ← App Service / Container Apps
│   │   └── monitoring/               ← Application Insights, Log Analytics
│   ├── environments/
│   │   ├── dev/
│   │   ├── staging/
│   │   └── prod/
│   └── main.tf
│
└── docs/                             ← Specs + Architecture
    ├── specs/
    └── architecture/
```

---

## CI/CD Pipeline

```
┌──────────┐    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│  Commit   │───▶│  Gate 2:     │───▶│  Gate 3:     │───▶│  Deploy      │
│  + Push   │    │  Automated   │    │  Review      │    │  (if main)   │
└──────────┘    │              │    │              │    │              │
                │ • dotnet build│    │ • PR < 400   │    │ • Staging    │
                │ • dotnet test │    │   lines      │    │ • Smoke test │
                │ • ng test     │    │ • Senior     │    │ • Prod       │
                │ • ng build    │    │   approval   │    │              │
                │ • pytest      │    │ • Spec ref   │    └──────────────┘
                │ • SAST scan   │    │   verified   │
                │ • SCA scan    │    └──────────────┘
                │ • Coverage    │
                └──────────────┘
```

```yaml
# .github/workflows/ci.yml
name: CI Pipeline

on: [push, pull_request]

jobs:
  backend-tests:
    runs-on: ubuntu-latest
    services:
      sqlserver:
        image: mcr.microsoft.com/mssql/server:2022-latest
        env:
          ACCEPT_EULA: Y
          MSSQL_SA_PASSWORD: YourStr0ngP@ssword
        ports: ['1433:1433']
      redis:
        image: redis:7-alpine
        ports: ['6379:6379']
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-dotnet@v4
        with:
          dotnet-version: '8.0.x'
      - run: dotnet restore backend/AccountPlanning.sln
      - run: dotnet build backend/AccountPlanning.sln --no-restore
      - run: dotnet test backend/AccountPlanning.sln --no-build --collect:"XPlat Code Coverage"
        # Gate 2: coverage threshold

  frontend-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: cd frontend && npm ci
      - run: cd frontend && npx ng test --watch=false --browsers=ChromeHeadless
      - run: cd frontend && npx ng build --configuration production

  ai-engine-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.12'
      - run: cd ai-engine && pip install -r requirements.txt
      - run: cd ai-engine && pytest

  security-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: SAST Scan
        uses: github/codeql-action/analyze@v3
      - name: SCA Scan (backend)
        run: dotnet list backend/AccountPlanning.sln package --vulnerable --include-transitive
      - name: SCA Scan (frontend)
        run: cd frontend && npm audit --audit-level=high
      - name: SCA Scan (ai-engine)
        run: cd ai-engine && pip-audit
```

---

## Escalabilidad

```
                    Bajo uso              Alto uso
                    ─────────             ──────────
Frontend:           1 instance (Static)   CDN (Azure Front Door)
Backend (.NET):     2 instances           6 instances (auto-scale CPU > 70%)
AI Engine:          1 instance            3 instances (auto-scale queue depth)
SQL Server:         Primary only          Primary + read replica
Redis:              Basic tier            Standard tier (replication)
```

| Componente | Bottleneck esperado | Solucion |
|-----------|-------------------|----------|
| AI Engine | Claude API latency (2-4s) | Queue + async, cache de analisis similares |
| SQL Server | Dashboard queries con joins | Read replicas, IMemoryCache + Redis, materialized views |
| Hangfire jobs | Muchas oportunidades a re-evaluar | Concurrencia limitada, batch processing |
| Angular bundle | Tamano inicial de carga | Lazy loading por feature module, preloading strategy |

---

*Deployment e infraestructura v2.0 — Account Planning Inteligente — Tech&Solve — Stack: Angular + .NET 8 + Python (AI) + SQL Server*
