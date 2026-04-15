# Prompts Estructurados — Spec Agent Portal

Prompts listos para usar que invocan las specs L2 aprobadas del Spec Agent Portal. Cada prompt sigue el template del framework AI-Driven Engineering y esta trazado a una spec + skill.

## Indice — Backend (.NET 10)

| Prompt | Spec | Skill | Tarea |
|--------|------|-------|-------|
| [PROMPT-01](PROMPT-01-manage-projects.md) | SPEC-L2-manage-projects v1.0.0 | SK-001 | Implementar Governance Service |
| [PROMPT-02](PROMPT-02-create-specs.md) | SPEC-L2-create-specs v1.0.0 | SK-001 | Implementar Spec Management + Agent Orchestration |
| [PROMPT-03](PROMPT-03-review-specs.md) | SPEC-L2-review-specs v1.0.0 | SK-001 | Implementar Review Service |
| [PROMPT-04](PROMPT-04-generate-prompts.md) | SPEC-L2-generate-prompts v1.0.0 | SK-001 | Implementar Prompt Builder |
| [PROMPT-05](PROMPT-05-metrics-dashboard.md) | SPEC-L2-metrics-dashboard v1.0.0 | SK-001 | Implementar Metrics Service |
| [PROMPT-06](PROMPT-06-manage-skills.md) | SPEC-L2-manage-skills v1.0.0 | SK-001 | Implementar Skill Catalog Service |
| [PROMPT-07](PROMPT-07-sync-devops.md) | SPEC-L2-sync-devops v1.0.0 | SK-001 | Implementar DevOps Sync Service |
| [PROMPT-08](PROMPT-08-data-masking.md) | SPEC-L1 (Clasificacion datos) | SK-006 | Implementar Data Masking (logs, API, ambientes) |
| [PROMPT-09](PROMPT-09-dlp-guard-llm.md) | SPEC-L2-create-specs v1.0.0 | SK-007 | Implementar DLP Guard LLM (pre/post-prompt) |

## Indice — Frontend (Angular 21)

| Prompt | Spec | Pareo Backend | Tarea |
|--------|------|---------------|-------|
| [PROMPT-00-FE](PROMPT-00-FE-ARCHITECTURE.md) | SPEC-L1-spec-agent-portal v1.0.0 | — (base) | Scaffold Angular 21 + routing + core + shared + auth |
| [PROMPT-01-FE](PROMPT-01-FE-projects.md) | SPEC-L2-manage-projects v1.0.0 | PROMPT-01 | Modulo Projects (dashboard, CRUD, ADRs, quality gates) |
| [PROMPT-02-FE](PROMPT-02-FE-create-specs.md) | SPEC-L2-create-specs v1.0.0 | PROMPT-02 | Modulo Spec Editor (wizard, Monaco, generacion IA) |
| [PROMPT-03-FE](PROMPT-03-FE-review-specs.md) | SPEC-L2-review-specs v1.0.0 | PROMPT-03 | Modulo Review (panel, validacion, comments, decision) |
| [PROMPT-04-FE](PROMPT-04-FE-generate-prompts.md) | SPEC-L2-generate-prompts v1.0.0 | PROMPT-04 | Modulo Prompt Builder (selector, preview, copy, historial) |
| [PROMPT-05-FE](PROMPT-05-FE-metrics-dashboard.md) | SPEC-L2-metrics-dashboard v1.0.0 | PROMPT-05 | Modulo Metrics (charts, trazabilidad, export) |
| [PROMPT-06-FE](PROMPT-06-FE-manage-skills.md) | SPEC-L2-manage-skills v1.0.0 | PROMPT-06 | Modulo Skill Catalog (CRUD, metricas, deprecacion) |

## Uso

1. Verificar que la spec esta APPROVED (Gate 1 pasado)
2. Adjuntar el contenido de la spec como contexto
3. Adjuntar los ADRs referenciados
4. Copiar el prompt y usarlo con la herramienta de IA preferida
5. Revisar el output generado antes de hacer PR

## Orden recomendado de implementacion

Backend y frontend se ejecutan **en paralelo** por cada caso de uso. El scaffold frontend (PROMPT-00-FE) se ejecuta una sola vez al inicio.

```
Paso 0: PROMPT-00-FE (scaffold Angular — una sola vez)

Paso 1: PROMPT-01 (backend) + PROMPT-01-FE (frontend) — en paralelo
Paso 2: PROMPT-02 (backend) + PROMPT-02-FE (frontend) — en paralelo
Paso 3: PROMPT-03 (backend) + PROMPT-03-FE (frontend) — en paralelo
Paso 4: PROMPT-04 (backend) + PROMPT-04-FE (frontend) — en paralelo
Paso 5: PROMPT-06 (backend) + PROMPT-06-FE (frontend) — en paralelo
Paso 6: PROMPT-05 (backend) + PROMPT-05-FE (frontend) — en paralelo
Paso 7: PROMPT-07 (backend solo — sin UI)
Paso 8: PROMPT-08 (Data Masking — SK-006) — transversal, aplica a todo el backend
Paso 9: PROMPT-09 (DLP Guard LLM — SK-007) — middleware DLP para integracion con Claude API
```

## Dependencias

- PROMPT-00-FE debe ejecutarse ANTES de cualquier PROMPT-XX-FE
- PROMPT-02 requiere PROMPT-01 (proyectos son la base)
- PROMPT-03 requiere PROMPT-02 (revision necesita specs)
- PROMPT-04 requiere PROMPT-02 + PROMPT-03 (prompt builder usa specs aprobadas)
- PROMPT-07 requiere PROMPT-03 (sync se dispara al aprobar)
- PROMPT-05 y PROMPT-06 son independientes entre si
