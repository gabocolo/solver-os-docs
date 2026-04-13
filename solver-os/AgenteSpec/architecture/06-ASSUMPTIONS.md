# 06 — Supuestos, Riesgos y Checklist

| Campo | Valor |
|-------|-------|
| **Proyecto** | Spec Agent Portal |
| **Version** | 1.0.0 |
| **Fecha** | 2026-04-09 |

---

## Supuestos

### Infraestructura
| # | Supuesto | Impacto si es falso |
|---|----------|-------------------|
| S-01 | Se dispone de una VM con 4 vCPU / 8 GB RAM para el MVP | Redimensionar infra o usar cloud managed services |
| S-02 | Azure Service Bus esta disponible (suscripcion Azure activa) | Usar RabbitMQ local como alternativa |
| S-03 | Se tiene acceso a la Claude API de Anthropic con creditos suficientes | Bloquea toda generacion con IA — CRUD manual sigue operativo |
| S-04 | PostgreSQL 16 y Redis 7 corren como containers Docker | Usar servicios cloud managed si no hay Docker disponible |

### Integraciones
| # | Supuesto | Impacto si es falso |
|---|----------|-------------------|
| S-05 | La organizacion Azure DevOps `juancolorado0526/IA Automatizaciones` existe y es accesible | Sync con DevOps no funciona — specs se aprueban igual |
| S-06 | Se puede generar un PAT con permisos de Work Items (Read, Write) | Sync con DevOps bloqueado hasta tener PAT |
| S-07 | OAuth via GitHub o Azure AD esta configurado para el dominio | Login no funciona — requiere configuracion previa |

### Organizacion
| # | Supuesto | Impacto si es falso |
|---|----------|-------------------|
| S-08 | El equipo adopta el framework AI-Driven Engineering v1.1 | El portal no tiene sentido sin el marco de gobernanza |
| S-09 | Hay al menos 1 ARCHITECT y 1 SENIOR_DEV en el equipo | Flujos de aprobacion requieren roles separados |
| S-10 | Los ADRs del framework (001-006) estan vigentes y aprobados | Las validaciones de SK-004 referencian ADRs que no existirian |

### Tecnico
| # | Supuesto | Impacto si es falso |
|---|----------|-------------------|
| S-11 | Claude Opus genera specs L1 de calidad aceptable con el template proporcionado | Requiere iteracion de prompt engineering |
| S-12 | Claude Sonnet genera specs L2/L3 de calidad aceptable | Requiere iteracion o upgrade a Opus (mayor costo) |
| S-13 | La latencia de la Claude API es < 120s para L1 y < 60s para L2/L3 | Ajustar timeouts y comunicar al usuario |
| S-14 | El DLP filter detecta patrones comunes de PII (email, documento, telefono) | Patrones no detectados se filtran en revision humana |

---

## Riesgos

| # | Riesgo | Probabilidad | Impacto | Mitigacion |
|---|--------|-------------|---------|------------|
| R-01 | Claude API no disponible (downtime) | Media | Alto | Circuit breaker + CRUD manual como fallback |
| R-02 | Calidad de generacion insuficiente (alucinaciones) | Media | Alto | Templates bien definidos + revision humana obligatoria |
| R-03 | Costo de tokens excede presupuesto | Media | Medio | Monitoreo de tokens, limites por proyecto, modelo correcto por tarea |
| R-04 | Azure DevOps API cambia (breaking change) | Baja | Medio | Usar version pinneada (v7.1), adapter aislado (ADR-001) |
| R-05 | PII se filtra en logs o respuestas | Baja | Critico | DLP pre/post, scan en pipeline, revision humana |
| R-06 | Adopcion baja del equipo | Media | Alto | Piloto con 1 equipo, metricas de mejora, capacitacion |
| R-07 | Performance del dashboard con muchas specs | Baja | Bajo | Cache Redis, indices PostgreSQL, paginacion |
| R-08 | Prompt injection en input del usuario | Baja | Alto | Validacion de input, DLP filter, no ejecutar codigo generado |
| R-09 | PAT de Azure DevOps expuesto | Baja | Critico | Cifrado en vault, nunca en logs, rotacion periodica |
| R-10 | Conflictos de versionado semver en specs | Baja | Bajo | Validacion automatica en SK-004 |

---

## Dependencias externas

| Dependencia | Tipo | Criticidad | Fallback |
|-------------|------|-----------|----------|
| Claude API (Anthropic) | SaaS | Alta | CRUD manual sin IA |
| Azure Service Bus | PaaS | Media | RabbitMQ local |
| Azure DevOps API | SaaS | Baja | Portal sin sync (specs operan independiente) |
| GitHub/Azure AD OAuth | SaaS | Alta | Modo desarrollo con auth bypass (solo dev) |

---

## Checklist de arranque

### Pre-requisitos de infraestructura
- [ ] VM o host con Docker instalado (Docker Engine 24+)
- [ ] Docker Compose v2 disponible
- [ ] Acceso a internet desde el host (para pull de imagenes y APIs externas)
- [ ] Puerto 4200 (frontend), 5000 (backend), 5432 (postgres), 5341 (seq) disponibles
- [ ] Suscripcion Azure con Azure Service Bus namespace creado

### Pre-requisitos de servicios externos
- [ ] Cuenta Anthropic con API key y creditos
- [ ] Organizacion Azure DevOps creada con proyecto destino
- [ ] PAT de Azure DevOps con scope Work Items (Read, Write)
- [ ] App OAuth registrada (GitHub o Azure AD) con redirect URI configurado

### Pre-requisitos de gobernanza
- [ ] ADRs 001-006 del framework revisados y vigentes
- [ ] Estandares de codigo (CODE-STANDARDS.md) definidos
- [ ] Roles del equipo asignados (ARCHITECT, LEAD, SENIOR_DEV, QA)
- [ ] Quality gates definidos (Gate 1, 2, 3)

### Setup inicial
- [ ] Clonar repositorio
- [ ] Crear `.env` a partir de `.env.example` con credenciales reales
- [ ] Ejecutar `docker-compose up -d`
- [ ] Verificar health check: `curl http://localhost:5000/health`
- [ ] Ejecutar migraciones: `docker-compose exec backend dotnet ef database update`
- [ ] Crear usuario administrador inicial (seed)
- [ ] Configurar proyecto inicial con ADRs y quality gates
- [ ] Verificar conexion con Azure DevOps (configurar en UI)
- [ ] Ejecutar smoke test: crear proyecto → crear spec manual → enviar a revision

### Validacion post-setup
- [ ] Login via OAuth funciona
- [ ] Crear proyecto funciona (201 Created)
- [ ] Generar spec L1 con IA funciona (202 → WebSocket → borrador)
- [ ] Enviar spec a revision funciona (validacion SK-004 pasa)
- [ ] Dashboard de metricas carga (< 3s)
- [ ] Logs visibles en Seq (http://localhost:8080)
- [ ] No hay PII en logs de Seq
