# Prompts Estructurados — Spec Agent Portal

Prompts listos para usar que invocan las specs L2 aprobadas del Spec Agent Portal. Cada prompt sigue el template del framework AI-Driven Engineering y esta trazado a una spec + skill.

## Indice

| Prompt | Spec | Skill | Tarea |
|--------|------|-------|-------|
| [PROMPT-01](PROMPT-01-manage-projects.md) | SPEC-L2-manage-projects v1.0.0 | SK-001 | Implementar Governance Service |
| [PROMPT-02](PROMPT-02-create-specs.md) | SPEC-L2-create-specs v1.0.0 | SK-001 | Implementar Spec Management + Agent Orchestration |
| [PROMPT-03](PROMPT-03-review-specs.md) | SPEC-L2-review-specs v1.0.0 | SK-001 | Implementar Review Service |
| [PROMPT-04](PROMPT-04-generate-prompts.md) | SPEC-L2-generate-prompts v1.0.0 | SK-001 | Implementar Prompt Builder |
| [PROMPT-05](PROMPT-05-metrics-dashboard.md) | SPEC-L2-metrics-dashboard v1.0.0 | SK-001 | Implementar Metrics Service |
| [PROMPT-06](PROMPT-06-manage-skills.md) | SPEC-L2-manage-skills v1.0.0 | SK-001 | Implementar Skill Catalog Service |
| [PROMPT-07](PROMPT-07-sync-devops.md) | SPEC-L2-sync-devops v1.0.0 | SK-001 | Implementar DevOps Sync Service |

## Uso

1. Verificar que la spec esta APPROVED (Gate 1 pasado)
2. Adjuntar el contenido de la spec como contexto
3. Adjuntar los ADRs referenciados
4. Copiar el prompt y usarlo con la herramienta de IA preferida
5. Revisar el output generado antes de hacer PR

## Orden recomendado de implementacion

```
1. PROMPT-01 (Projects + Governance) — base del sistema
2. PROMPT-02 (Create Specs) — funcionalidad core
3. PROMPT-03 (Review Specs) — flujo de aprobacion
4. PROMPT-04 (Generate Prompts) — prompt builder
5. PROMPT-06 (Skills) — catalogo
6. PROMPT-05 (Metrics) — dashboard
7. PROMPT-07 (DevOps Sync) — integracion externa
```
