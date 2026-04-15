# /tests — Pruebas del Spec Agent Portal

## Escenarios GWT por Spec L2

| Spec L2 | Caso de uso | Tests | Alta | Media | Baja |
|---------|-------------|-------|------|-------|------|
| SPEC-L2-manage-projects | ManageProjectsAndGovernance | 6 | 3 | 2 | 1 |
| SPEC-L2-create-specs | CreateSpecWithAI | 8 | 4 | 3 | 1 |
| SPEC-L2-review-specs | ReviewAndApproveSpec | 8 | 4 | 3 | 1 |
| SPEC-L2-generate-prompts | GenerateStructuredPrompt | 6 | 3 | 2 | 1 |
| SPEC-L2-metrics-dashboard | MetricsDashboard | 6 | 2 | 3 | 1 |
| SPEC-L2-manage-skills | ManageSkillCatalog | 6 | 3 | 2 | 1 |
| SPEC-L2-sync-devops | SyncSpecToDevOps | 8 | 4 | 3 | 1 |
| **Total** | | **48** | **23** | **18** | **7** |

## Estructura

| Carpeta | Tipo | Stack |
|---------|------|-------|
| unit/ | Tests unitarios GWT | xUnit + NSubstitute (backend), Jasmine/Karma (frontend) |
| integration/ | Tests de integracion | xUnit + TestContainers (PostgreSQL) |
| contract/ | Tests de contrato OpenAPI | Validacion de contratos contra implementacion |
| e2e/ | Tests end-to-end | Playwright o similar |
| fixtures/ | Datos sinteticos | JSON con datos de prueba (NUNCA PII real) |

## Reglas

- Tests se crean **ANTES** de la implementacion (SK-002)
- Datos sinteticos siempre — nunca PII real
- Cobertura: 100% de escenarios de la spec L2
- Se ejecutan con SK-005 (pipeline de validacion PR)
