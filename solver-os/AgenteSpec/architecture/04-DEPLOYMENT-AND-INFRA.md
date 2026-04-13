# 04 — Deployment e Infraestructura

| Campo | Valor |
|-------|-------|
| **Proyecto** | Spec Agent Portal |
| **Version** | 1.0.0 |
| **Fecha** | 2026-04-09 |

---

## Arquitectura de deployment (MVP)

```
┌──────────────────────────────────────────────────────────────────────┐
│                        VM / Host (Docker)                            │
│                                                                      │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────────┐  │
│  │   Frontend       │  │    Backend       │  │   Worker            │  │
│  │   (nginx + SPA)  │  │    (.NET 10 API) │  │   (.NET 10 Worker) │  │
│  │   :4200          │  │    :5000         │  │   (no port)        │  │
│  └────────┬─────────┘  └────────┬─────────┘  └────────┬───────────┘  │
│           │                     │                      │              │
│           │              ┌──────▼──────────────────────▼───────┐      │
│           │              │         Azure Service Bus           │      │
│           │              │     (cloud — queue de jobs IA)      │      │
│           │              └────────────────────────────────────┘      │
│           │                     │                                     │
│  ┌────────▼─────────────────────▼────────────────────────────────┐   │
│  │                    Docker Network (internal)                    │   │
│  └──────┬──────────────────┬──────────────────┬──────────────────┘   │
│         │                  │                  │                       │
│  ┌──────▼──────┐    ┌──────▼──────┐    ┌──────▼──────┐               │
│  │ PostgreSQL  │    │   Redis     │    │    Seq      │               │
│  │   16        │    │    7        │    │  (logs)     │               │
│  │  :5432      │    │  :6379      │    │  :5341      │               │
│  └─────────────┘    └─────────────┘    └─────────────┘               │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
                           │
                    ┌──────▼──────────┐
                    │  Azure DevOps   │
                    │  REST API v7.1  │
                    └─────────────────┘
                           │
                    ┌──────▼──────────┐
                    │  Claude API     │
                    │  (Anthropic)    │
                    └─────────────────┘
```

---

## Docker Compose

```yaml
# docker-compose.yml
version: "3.9"

services:
  # --- Frontend ---
  frontend:
    build:
      context: ./src/frontend
      dockerfile: Dockerfile
    ports:
      - "4200:80"
    depends_on:
      - backend
    environment:
      - API_URL=http://backend:5000
    restart: unless-stopped

  # --- Backend API ---
  backend:
    build:
      context: ./src/backend
      dockerfile: Dockerfile
    ports:
      - "5000:5000"
    depends_on:
      - postgres
      - redis
    environment:
      - ConnectionStrings__Default=Host=postgres;Database=specagent;Username=specagent;Password=${DB_PASSWORD}
      - ConnectionStrings__Redis=redis:6379
      - ServiceBus__ConnectionString=${SERVICEBUS_CONNECTION}
      - LLM__Provider=Anthropic
      - LLM__ApiKey=${CLAUDE_API_KEY}
      - LLM__ModelL1=claude-opus-4-6
      - LLM__ModelL2=claude-sonnet-4-6
      - LLM__ModelValidation=claude-haiku-4-5-20251001
      - Auth__Provider=${AUTH_PROVIDER}
      - Auth__ClientId=${AUTH_CLIENT_ID}
      - Auth__ClientSecret=${AUTH_CLIENT_SECRET}
      - Seq__Url=http://seq:5341
    restart: unless-stopped

  # --- Worker (async jobs) ---
  worker:
    build:
      context: ./src/backend
      dockerfile: Dockerfile.worker
    depends_on:
      - postgres
      - redis
    environment:
      - ConnectionStrings__Default=Host=postgres;Database=specagent;Username=specagent;Password=${DB_PASSWORD}
      - ConnectionStrings__Redis=redis:6379
      - ServiceBus__ConnectionString=${SERVICEBUS_CONNECTION}
      - LLM__Provider=Anthropic
      - LLM__ApiKey=${CLAUDE_API_KEY}
      - LLM__ModelL1=claude-opus-4-6
      - LLM__ModelL2=claude-sonnet-4-6
      - Seq__Url=http://seq:5341
    restart: unless-stopped

  # --- PostgreSQL ---
  postgres:
    image: postgres:16-alpine
    ports:
      - "5432:5432"
    environment:
      - POSTGRES_DB=specagent
      - POSTGRES_USER=specagent
      - POSTGRES_PASSWORD=${DB_PASSWORD}
    volumes:
      - pgdata:/var/lib/postgresql/data
      - ./scripts/init.sql:/docker-entrypoint-initdb.d/init.sql
    restart: unless-stopped

  # --- Redis ---
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    restart: unless-stopped

  # --- Seq (logs) ---
  seq:
    image: datalust/seq:latest
    ports:
      - "5341:5341"
      - "8080:80"
    environment:
      - ACCEPT_EULA=Y
    volumes:
      - seqdata:/data
    restart: unless-stopped

volumes:
  pgdata:
  seqdata:
```

---

## Variables de entorno (.env)

```bash
# Database
DB_PASSWORD=<secret>

# Azure Service Bus
SERVICEBUS_CONNECTION=<connection-string>

# Claude API (Anthropic)
CLAUDE_API_KEY=<api-key>

# OAuth
AUTH_PROVIDER=github  # o azure_ad
AUTH_CLIENT_ID=<client-id>
AUTH_CLIENT_SECRET=<client-secret>

# Azure DevOps (por proyecto, almacenado cifrado en DB)
# No va en .env — se configura por UI
```

**IMPORTANTE**: el archivo `.env` NUNCA se commitea al repositorio. Usar `.env.example` con valores placeholder.

---

## Estructura del repositorio

```
spec-agent-portal/
├── .github/                         # o .azure-pipelines/
│   └── workflows/
│       ├── ci.yml                   # Build + tests + SAST + SCA + DLP
│       └── cd.yml                   # Deploy a staging/prod
├── docs/
│   ├── architecture/                # Estos documentos
│   └── specs/                       # Specs versionadas
├── scripts/
│   ├── init.sql                     # DDL inicial
│   └── seed-dev.sql                 # Datos sinteticos para desarrollo
├── src/
│   ├── frontend/
│   │   ├── src/
│   │   │   ├── app/
│   │   │   │   ├── core/            # Auth, guards, interceptors
│   │   │   │   ├── shared/          # Componentes compartidos
│   │   │   │   ├── features/
│   │   │   │   │   ├── projects/    # Gestion de proyectos
│   │   │   │   │   ├── specs/       # CRUD + editor de specs
│   │   │   │   │   ├── reviews/     # Flujo de revision
│   │   │   │   │   ├── prompts/     # Prompt builder
│   │   │   │   │   ├── metrics/     # Dashboard
│   │   │   │   │   ├── skills/      # Catalogo de skills
│   │   │   │   │   └── devops/      # Config Azure DevOps
│   │   │   │   └── app.routes.ts
│   │   │   ├── environments/
│   │   │   └── styles/
│   │   ├── Dockerfile
│   │   ├── angular.json
│   │   └── package.json
│   └── backend/
│       ├── SpecAgent.Domain/         # Entidades, puertos (interfaces)
│       │   ├── Entities/
│       │   ├── Ports/
│       │   ├── Enums/
│       │   └── ValueObjects/
│       ├── SpecAgent.Application/    # Casos de uso, servicios
│       │   ├── Specs/
│       │   ├── Governance/
│       │   ├── Reviews/
│       │   ├── Prompts/
│       │   ├── Metrics/
│       │   ├── Skills/
│       │   └── DevOpsSync/
│       ├── SpecAgent.Infrastructure/ # Adaptadores
│       │   ├── Persistence/          # EF Core + PostgreSQL
│       │   ├── LLM/                  # Semantic Kernel + Claude
│       │   ├── DLP/                  # Filtros DLP
│       │   ├── Cache/                # Redis
│       │   ├── Messaging/            # Azure Service Bus
│       │   ├── DevOps/               # Azure DevOps REST client
│       │   └── Auth/                 # OAuth middleware
│       ├── SpecAgent.API/            # Minimal APIs, controllers
│       │   ├── Endpoints/
│       │   ├── Middleware/
│       │   └── Program.cs
│       ├── SpecAgent.Worker/         # Worker de jobs async
│       │   ├── Handlers/
│       │   └── Program.cs
│       ├── SpecAgent.Tests/          # Tests unitarios + integracion
│       ├── Dockerfile
│       ├── Dockerfile.worker
│       └── SpecAgent.sln
├── docker-compose.yml
├── docker-compose.override.yml      # Overrides para dev local
├── .env.example
├── .gitignore
└── README.md
```

---

## Pipeline CI/CD

### CI (en cada PR)

```yaml
# ci.yml
trigger:
  - main
  - feature/*

stages:
  - stage: Build
    jobs:
      - job: Backend
        steps:
          - dotnet restore
          - dotnet build --no-restore
          - dotnet test --no-build --collect:"XPlat Code Coverage"
            # Gate: cobertura >= 80%

      - job: Frontend
        steps:
          - npm ci
          - npm run lint
          - npm run test -- --watch=false --code-coverage
          - npm run build -- --configuration=production

  - stage: Security
    jobs:
      - job: SAST
        steps:
          - run: dotnet tool install --global security-scan
          - run: security-scan ./src/backend/

      - job: SCA
        steps:
          - run: dotnet list package --vulnerable
          - run: npm audit --production

      - job: DLP_Scan
        steps:
          - run: detect-secrets scan --baseline .secrets.baseline
          - run: python scripts/scan-pii.py --diff HEAD~1
            # Gate: 0 hallazgos criticos

  - stage: ContractTests
    jobs:
      - job: APIContracts
        steps:
          - run: dotnet test --filter "Category=Contract"
```

### CD (merge a main)

```yaml
# cd.yml
trigger:
  branches:
    include: [main]

stages:
  - stage: Deploy_Staging
    jobs:
      - job: Deploy
        steps:
          - docker-compose build
          - docker-compose push
          - ssh deploy@staging "docker-compose pull && docker-compose up -d"

  - stage: Smoke_Tests
    dependsOn: Deploy_Staging
    jobs:
      - job: Smoke
        steps:
          - run: curl -f https://staging.specagent.internal/health
          - run: npm run test:e2e -- --env=staging

  - stage: Deploy_Production
    dependsOn: Smoke_Tests
    condition: succeeded()
    jobs:
      - job: Deploy
        steps:
          - docker-compose -f docker-compose.prod.yml build
          - docker-compose -f docker-compose.prod.yml push
          # Deploy manual o con aprobacion
```

---

## Capacidad del MVP

| Recurso | Especificacion |
|---------|---------------|
| VM | 4 vCPU, 8 GB RAM |
| Disco | 50 GB SSD |
| Usuarios concurrentes | 50 |
| Specs estimadas | hasta 1,000 |
| Backups | Diarios automaticos (pg_dump + cron) |

---

## Escalamiento futuro (post-MVP)

| Aspecto | MVP | Futuro |
|---------|-----|--------|
| Deployment | Docker Compose en VM | Kubernetes (AKS) |
| Base de datos | PostgreSQL single | PostgreSQL con replicas de lectura |
| Cache | Redis single | Redis Cluster |
| Workers | 1 instancia | Auto-scaling por profundidad de cola |
| CDN | Ninguno | Azure CDN para frontend |
| Secrets | .env cifrado | Azure Key Vault |
