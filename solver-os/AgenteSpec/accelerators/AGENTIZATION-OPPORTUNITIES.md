# Oportunidades de agentizacion — Spec Agent Portal

Analisis de procesos candidatos a ser automatizados mediante agentes de IA dentro del contexto del Spec Agent Portal y el framework AI-Driven Engineering.

---

## Matriz de evaluacion

| # | Proceso | Frecuencia | Esfuerzo actual (hrs/sem) | Repetitividad (1-5) | Cross-team (1-5) | Potencial automatizacion (1-5) | Score | Prioridad |
|---|---------|-----------|--------------------------|---------------------|-------------------|-------------------------------|-------|-----------|
| 1 | Generacion de spec L1 desde input de negocio | Semanal | 4-8 hrs | 4 | 5 | 5 | **14** | **Alta** |
| 2 | Derivacion de spec L2 desde L1 aprobada | Semanal | 2-4 hrs | 5 | 5 | 5 | **15** | **Alta** |
| 3 | Validacion automatica de estructura de spec (SK-004) | Diario | 1-2 hrs | 5 | 5 | 5 | **15** | **Alta** |
| 4 | Generacion de prompts estructurados desde spec | Diario | 0.5-1 hr | 5 | 5 | 5 | **15** | **Alta** |
| 5 | Sincronizacion spec → work item Azure DevOps | Semanal | 1-2 hrs | 5 | 4 | 5 | **14** | **Alta** |
| 6 | Generacion de tests GWT desde spec L2 | Semanal | 2-3 hrs | 4 | 4 | 4 | **12** | **Alta** |
| 7 | Revision de PII en codigo/prompts (DLP scan) | Diario | 1 hr | 5 | 5 | 4 | **14** | **Alta** |
| 8 | Generacion de contrato OpenAPI desde spec L2 | Semanal | 1-2 hrs | 4 | 3 | 4 | **11** | **Media** |
| 9 | Generacion de ADR asistido | Mensual | 2-4 hrs | 3 | 3 | 4 | **10** | **Media** |
| 10 | Calculo y reporte de metricas de framework | Semanal | 2 hrs | 4 | 4 | 3 | **11** | **Media** |
| 11 | Deteccion de specs obsoletas o sin actividad | Mensual | 1 hr | 3 | 3 | 3 | **9** | **Media** |
| 12 | Sugerencia de division de PRs > 400 lineas | Semanal | 0.5 hr | 3 | 3 | 3 | **9** | **Media** |
| 13 | Generacion automatica de changelog desde specs | Quincenal | 1 hr | 3 | 2 | 3 | **8** | **Media** |
| 14 | Analisis de coherencia entre L1 y L2 derivadas | Semanal | 1 hr | 3 | 3 | 2 | **8** | **Media** |
| 15 | Notificacion proactiva de skills con bajo rendimiento | Mensual | 0.5 hr | 3 | 2 | 2 | **7** | **Baja** |

---

## Agentes implementados en el MVP

Los procesos de alta prioridad ya estan implementados como funcionalidades del portal:

### Agente 1: Spec Generator Agent
- **Proceso**: #1 y #2 (Generacion de specs L1/L2/L3)
- **Tipo**: Agente de generacion con LLM
- **Como funciona**: Recibe input en lenguaje natural + contexto (ADRs, templates, parent spec), envia a Claude (Opus para L1, Sonnet para L2/L3) via Service Bus, retorna borrador estructurado con DLP bidireccional
- **Impacto**: 4-8 hrs → < 30 min por spec L1

### Agente 2: Spec Validator Agent (SK-004)
- **Proceso**: #3 (Validacion automatica)
- **Tipo**: Agente de validacion basado en reglas
- **Como funciona**: Ejecuta 10 validaciones automaticas al enviar spec a revision (campos, semver, clasificacion, parent, ADRs)
- **Impacto**: 1-2 hrs de revision manual eliminadas por cada spec

### Agente 3: Prompt Builder Agent
- **Proceso**: #4 (Generacion de prompts)
- **Tipo**: Agente de ensamblaje (sin LLM)
- **Como funciona**: Ensambla prompt estructurado desde spec + skill + ADRs + reglas DLP automaticas
- **Impacto**: Proceso manual de 30-60 min → < 2 segundos

### Agente 4: DevOps Sync Agent
- **Proceso**: #5 (Sincronizacion con Azure DevOps)
- **Tipo**: Agente de integracion event-driven
- **Como funciona**: Escucha evento SpecApproved, crea work item (Feature/User Story/Task), establece relaciones parent-child
- **Impacto**: 1-2 hrs manuales eliminadas, trazabilidad automatica

### Agente 5: DLP Guard Agent (SK-007)
- **Proceso**: #7 (Revision de PII)
- **Tipo**: Agente de seguridad basado en regex
- **Como funciona**: Sanitiza prompts pre-LLM y valida respuestas post-LLM buscando patrones de PII
- **Impacto**: Proteccion automatica en cada interaccion con LLM

---

## Oportunidades futuras (post-MVP)

### Agente 6: Test Generator Agent (SK-002)
- **Proceso**: #6
- **Prioridad**: Alta
- **Descripcion**: Genera tests GWT en codigo (xUnit/NUnit para .NET, Jest para Angular) a partir de los escenarios definidos en la spec L2
- **Estimacion de desarrollo**: 2-3 semanas
- **ROI**: 2-3 hrs/semana ahorradas por equipo

### Agente 7: Contract Generator Agent (SK-003)
- **Proceso**: #8
- **Prioridad**: Media
- **Descripcion**: Genera archivo OpenAPI/Swagger desde la seccion de contrato tecnico de la spec L2
- **Estimacion de desarrollo**: 1-2 semanas
- **ROI**: 1-2 hrs/semana ahorradas

### Agente 8: Coherence Analyzer Agent
- **Proceso**: #14
- **Prioridad**: Media
- **Descripcion**: Analiza automaticamente si las specs L2 derivadas son coherentes con la L1 parent (entidades referenciadas existen, reglas respetadas)
- **Estimacion de desarrollo**: 2 semanas
- **ROI**: Reduce ciclos de revision (CHANGES_REQUESTED) en ~30%

### Agente 9: Changelog Agent
- **Proceso**: #13
- **Prioridad**: Media
- **Descripcion**: Genera changelog automatico a partir de specs L3 aprobadas en un periodo
- **Estimacion de desarrollo**: 1 semana
- **ROI**: 1 hr/quincena ahorrada

### Agente 10: Spec Health Monitor Agent
- **Proceso**: #11 + #15
- **Prioridad**: Baja
- **Descripcion**: Detecta specs en DRAFT sin actividad > 2 semanas, skills con bajo rendimiento, y proyectos sin metricas recientes. Envia alertas proactivas
- **Estimacion de desarrollo**: 1 semana
- **ROI**: Gobernanza proactiva, no reactiva

---

## ROI consolidado del MVP

| Metrica | Sin portal (manual) | Con portal | Ahorro |
|---------|-------------------|-----------|--------|
| Crear spec L1 | 4-8 hrs | < 30 min | **~90%** |
| Derivar spec L2 | 2-4 hrs | < 15 min | **~90%** |
| Validar estructura | 1-2 hrs | < 10 seg (automatico) | **~100%** |
| Generar prompt | 30-60 min | < 2 seg | **~100%** |
| Sincronizar DevOps | 1-2 hrs | automatico | **~100%** |
| Verificar PII | 1 hr | automatico | **~100%** |
| **Total semanal por equipo** | **~12-20 hrs** | **~1-2 hrs** | **~85-90%** |

---

## Agentes transversales (aplican a multiples clientes)

| Agente | Tipo | Aplicabilidad | Estado |
|--------|------|--------------|--------|
| Spec Generator | Generacion IA | Universal | Implementado en MVP |
| Spec Validator (SK-004) | Validacion reglas | Universal | Implementado en MVP |
| Prompt Builder | Ensamblaje local | Universal | Implementado en MVP |
| DLP Guard (SK-007) | Seguridad regex | Universal | Implementado en MVP |
| DevOps Sync | Integracion | Azure DevOps (extensible a Jira, Linear) | MVP: Azure DevOps |
| Test Generator (SK-002) | Generacion IA | Universal | Futuro v2 |
| Contract Generator (SK-003) | Generacion IA | Universal | Futuro v2 |

**Clave**: los agentes transversales se construyen una vez y se reutilizan en multiples equipos y clientes. Invertir en su madurez beneficia a toda la organizacion.
