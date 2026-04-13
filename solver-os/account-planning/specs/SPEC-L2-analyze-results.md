# Especificacion de Sistema (Nivel 2)

| Campo | Valor |
|-------|-------|
| **Spec ID** | SPEC-L2-analyze-results |
| **Nivel** | L2 — Sistema |
| **Version** | 1.0.0 |
| **Parent L1** | SPEC-L1-account-planning v2.0.0 |
| **Estado** | DRAFT |
| **Creado por** | @juan.gabriel.colorado |
| **Fecha de creacion** | 2026-04-09 |
| **Aprobado por** | Pendiente |
| **Fecha de aprobacion** | Pendiente |
| **PR** | Pendiente |

---

## Descripcion general

Modulo de analisis e insights que toma los hallazgos crudos de una investigacion completada y genera un dashboard interactivo con 9 secciones. La IA interpreta, conecta y convierte la informacion en oportunidades comerciales reales, cruzando con el catalogo de servicios de Tech&Solve.

---

## Sistema

**Nombre:** Analysis & Insights Engine

## Caso de uso

**Nombre:** AnalyzeAndDisplayResults

**Descripcion:** Una vez que la investigacion esta completa (status COMPLETED), la IA analiza los hallazgos, identifica patrones, cruza con el portafolio de servicios de Tech&Solve y genera un dashboard interactivo con ficha del cliente, hallazgos clave, stakeholders con lectura comercial, oportunidades priorizadas, comparativa de sector, mapa dolor-valor-servicio, fuentes consultadas y alertas.

---

## Contexto

- Los hallazgos crudos vienen de la fase de investigacion (SPEC-L2-execute-research)
- La IA no debe repetir informacion literal — debe interpretarla, conectarla y convertirla en oportunidades
- El equipo comercial valora: identificar tensiones reales (crecimiento vs rentabilidad, legacy vs agilidad, regulacion vs experiencia)
- El mapa dolor-valor-servicio es el artefacto mas usado en reuniones — debe estar listo para copiar a PPT
- El comercial puede interactuar con el dashboard: marcar hallazgos como irrelevantes, agregar notas
- Si el cliente opera en otro idioma, se debe alertar para preparar materiales

---

## Contrato tecnico

### Input — GenerateAnalysis

| Campo | Tipo | Requerido | Descripcion |
|-------|------|-----------|------------|
| investigationId | string (UUID) | Si | ID de la investigacion completada |

### Input — UpdateFinding (interaccion del comercial)

| Campo | Tipo | Requerido | Descripcion |
|-------|------|-----------|------------|
| findingId | string (UUID) | Si | ID del hallazgo |
| isRelevant | boolean | No | Marcar como relevante (true) o irrelevante (false) |
| note | string | No | Nota del comercial sobre el hallazgo |

### Output — AnalysisResponse (Dashboard)

| Campo | Tipo | Descripcion |
|-------|------|------------|
| analysisId | string (UUID) | ID unico del analisis generado |
| investigationId | string (UUID) | ID de la investigacion fuente |
| clientCard | object | Ficha del cliente |
| clientCard.name | string | Nombre de la empresa |
| clientCard.industry | string | Sector |
| clientCard.country | string | Pais |
| clientCard.estimatedSize | string | Tamano estimado (SMALL, MEDIUM, LARGE, ENTERPRISE) |
| clientCard.summary | string | Resumen ejecutivo de la empresa (2-3 oraciones) |
| keyFindings | FindingCard[] | Hallazgos clave organizados en tarjetas |
| keyFindings[].findingId | string (UUID) | ID del hallazgo |
| keyFindings[].title | string | Titulo resumido del hallazgo |
| keyFindings[].content | string | Contenido del hallazgo |
| keyFindings[].source | string | Nombre de la fuente |
| keyFindings[].sourceUrl | string | URL de la fuente |
| keyFindings[].date | string (ISO 8601) | Fecha del hallazgo |
| keyFindings[].confidence | string | HIGH, MEDIUM, LOW |
| keyFindings[].type | string | NEWS, FINANCIAL, PUBLICATION, LINKEDIN, CASE_STUDY, SERVICE_MATCH |
| keyFindings[].isRelevant | boolean | Si el comercial lo marco como relevante (default true) |
| keyFindings[].note | string | Nota del comercial (null si no tiene) |
| keyFindings[].commercialInsight | string | Insight generado por la IA: que significa esto para el negocio y como se convierte en oportunidad |
| stakeholders | StakeholderAnalysis[] | Personas clave con lectura comercial |
| stakeholders[].stakeholderId | string (UUID) | ID del stakeholder |
| stakeholders[].name | string | Nombre |
| stakeholders[].role | string | Cargo |
| stakeholders[].type | string | STRATEGIC_DECISOR, TECH_DECISOR, INFLUENCER, GATEKEEPER, FINANCIAL_DECISOR |
| stakeholders[].relevance | string | Relevancia comercial |
| stakeholders[].painPoints | string[] | Dolores probables basados en su rol |
| stakeholders[].approach | string | Como abordarlo comercialmente |
| stakeholders[].linkedinUrl | string | URL del perfil de LinkedIn |
| opportunities | OpportunityCard[] | Oportunidades detectadas y priorizadas |
| opportunities[].opportunityId | string (UUID) | ID de la oportunidad |
| opportunities[].title | string | Titulo de la oportunidad |
| opportunities[].description | string | Descripcion detallada |
| opportunities[].horizon | string | SHORT_TERM, MEDIUM_TERM, LONG_TERM |
| opportunities[].matchedServices | string[] | Servicios de Tech&Solve que aplican |
| opportunities[].matchedCaseStudies | string[] | Casos de exito relevantes del RAG |
| opportunities[].priority | string | HIGH, MEDIUM, LOW |
| opportunities[].rationale | string | Por que la IA identifica esta oportunidad |
| sectorComparison | object | Comparativa del cliente en su sector |
| sectorComparison.position | string | Posicion estimada del cliente en el sector |
| sectorComparison.marketShare | string | Participacion de mercado estimada (si disponible) |
| sectorComparison.competitors | string[] | Competidores principales identificados |
| sectorComparison.trends | string[] | Tendencias del sector relevantes |
| sectorComparison.differentiators | string[] | Diferenciadores del cliente |
| painValueServiceMap | PainValueService[] | Tabla dolor-valor-servicio lista para presentacion |
| painValueServiceMap[].pain | string | Dolor identificado del cliente |
| painValueServiceMap[].value | string | Valor que Tech&Solve puede aportar |
| painValueServiceMap[].service | string | Servicio especifico del portafolio |
| sources | SourceReference[] | Todas las fuentes consultadas |
| sources[].name | string | Nombre de la fuente |
| sources[].url | string | URL |
| sources[].type | string | WEB, LINKEDIN, RAG, FINANCIAL |
| sources[].accessDate | string (ISO 8601) | Fecha de consulta |
| internationalAlert | object | Alerta si el cliente opera en otro idioma |
| internationalAlert.isInternational | boolean | Si el cliente opera internacionalmente |
| internationalAlert.languages | string[] | Idiomas detectados |
| internationalAlert.recommendation | string | Recomendacion (ej: "Preparar materiales en ingles") |
| decisionStructure | object | Estructura de decision del cliente |
| decisionStructure.model | string | Modelo de decision (comite, jerarquico, mixto) |
| decisionStructure.keyInfluencers | string | Quien influye mas en las decisiones |
| decisionStructure.approvalProcess | string | Como se aprueban compras de tecnologia/servicios |
| generatedAt | string (ISO 8601) | Timestamp de generacion del analisis |

---

## Comportamiento esperado

1. El sistema valida que la investigacion existe y esta en status COMPLETED
2. Recupera todos los hallazgos y stakeholders de la investigacion
3. Invoca el AIAnalysisEngine con los hallazgos como input
4. La IA genera:
   a. **Ficha del cliente** — datos basicos + resumen ejecutivo (no descriptivo, sino estrategico)
   b. **Insights por hallazgo** — para cada hallazgo, que significa para el negocio y como se convierte en oportunidad
   c. **Lectura comercial de stakeholders** — por cada stakeholder, dolores probables, como abordarlo, tipo de decisor
   d. **Oportunidades priorizadas** — cruzadas con el catalogo de servicios de Tech&Solve y casos de exito del RAG
   e. **Comparativa de sector** — posicion del cliente, tendencias, competidores
   f. **Mapa dolor-valor-servicio** — tabla de 3 columnas lista para presentacion
   g. **Estructura de decision** — como se toman decisiones de compra en esa empresa
   h. **Alerta internacional** — si opera en otro idioma
5. Persiste el analisis completo
6. Retorna el dashboard con todas las secciones
7. El comercial puede interactuar: marcar hallazgos como irrelevantes, agregar notas (UpdateFinding)

---

## Tipos de error

| Codigo de error | HTTP Status | Descripcion | Condicion de disparo |
|----------------|-------------|------------|---------------------|
| INVESTIGATION_NOT_FOUND | 404 | La investigacion no existe | investigationId no encontrado |
| INVESTIGATION_NOT_COMPLETED | 400 | La investigacion aun no ha terminado | status != COMPLETED |
| AI_ANALYSIS_FAILED | 500 | La IA no pudo generar el analisis | Error del modelo o timeout |
| FINDING_NOT_FOUND | 404 | El hallazgo no existe | findingId no encontrado en UpdateFinding |
| ANALYSIS_ALREADY_EXISTS | 409 | Ya existe un analisis para esta investigacion | Se intenta regenerar sin cambios |

---

## Dependencias permitidas

> **REGLA (ADR-004):** Solo estas dependencias pueden ser usadas en la implementacion.

| Dependencia | Proposito | Tipo de interfaz |
|------------|----------|-----------------|
| InvestigationRepository | Obtener hallazgos y stakeholders de la investigacion | Puerto (IInvestigationRepository) |
| AIAnalysisEngine | Motor de IA que genera insights, oportunidades y mapa dolor-valor-servicio | Puerto (IAIAnalysisEngine) |
| ServiceCatalog | Consultar catalogo de servicios de Tech&Solve para cruzar con oportunidades | Puerto (IServiceCatalog) |
| CaseStudyCatalog | Consultar casos de exito para match con oportunidades | Puerto (ICaseStudyCatalog) |
| AnalysisRepository | Persistir el analisis generado | Puerto (IAnalysisRepository) |
| EventBus | Publicar evento AnalysisGenerated | Puerto (IEventPublisher) |

---

## Requisitos de ejecucion

| Requisito | Valor | Notas |
|-----------|-------|-------|
| **Latencia maxima** | 30000 ms (30s) | Incluye llamada al modelo de IA para analisis |
| **Idempotencia** | No requerida | El analisis puede regenerarse con resultados diferentes |
| **Atomicidad** | Transaccion atomica | Analisis completo se persiste junto |
| **Observabilidad** | Ver abajo | |

### Observabilidad requerida

- **Logs:** Analisis iniciado (INFO), IA genero N oportunidades (INFO), N stakeholders analizados (INFO), error de IA (ERROR), hallazgo marcado irrelevante (INFO)
- **Trazas:** span por analisis completo, span por llamada a IA, span por consulta de catalogo de servicios
- **Metricas:** analysis_duration_ms, opportunities_per_analysis, pain_value_items_per_analysis, findings_marked_irrelevant_rate
- **Alertas:** AI analysis error rate > 5% en 5 min, analysis duration p95 > 45s

---

## Restricciones de arquitectura

| ADR | Titulo | Impacto en esta spec |
|-----|--------|---------------------|
| ADR-001 | Ports & Adapters | AnalyzeResults depende de 6 puertos. AIAnalysisEngine es un puerto — se puede cambiar el modelo sin afectar el core |
| ADR-004 | Dependency Management | Solo las 6 dependencias listadas |
| ADR-005 | Testing Strategy | Tests GWT antes de implementar. Mock del AIAnalysisEngine en tests |

---

## Restricciones y consideraciones

| Tipo | Descripcion |
|------|------------|
| **Limitaciones tecnicas** | La calidad del analisis depende de la calidad de los hallazgos. El catalogo de servicios debe estar actualizado en el RAG |
| **Integraciones necesarias** | Semantic Kernel para orquestar el agente de analisis |
| **Consideraciones de seguridad** | No enviar datos sensibles del prospecto a la IA sin sanitizar. No inferir informacion falsa sobre personas |
| **Consideraciones de rendimiento** | Analisis en < 30s. Cache del catalogo de servicios para evitar consultas repetidas |

---

## Criterios de aceptacion

- [ ] El dashboard muestra las 9 secciones con contenido relevante basado en los hallazgos
- [ ] Las oportunidades estan priorizadas y cruzadas con servicios reales de Tech&Solve
- [ ] Los stakeholders tienen lectura comercial: dolores probables y como abordarlos
- [ ] El mapa dolor-valor-servicio tiene minimo 3 filas con contenido accionable
- [ ] Los hallazgos incluyen un insight de la IA (no repeticion literal de la fuente)
- [ ] El comercial puede marcar hallazgos como irrelevantes
- [ ] Si el cliente opera en otro idioma, se muestra la alerta internacional
- [ ] El analisis se genera en menos de 30 segundos

---

## Escenarios de prueba (GWT)

> **REGLA (ADR-005):** Estos escenarios deben traducirse en tests ANTES de generar la implementacion.

| ID | Given (Dado que) | When (Cuando) | Then (Entonces) | Prioridad |
|----|-----------------|--------------|-----------------|-----------|
| TS-001 | Una investigacion COMPLETED con 15 hallazgos de web, LinkedIn y RAG | Se genera el analisis | Dashboard con las 9 secciones, oportunidades priorizadas, stakeholders con lectura comercial, mapa dolor-valor-servicio con >= 3 filas | Alta |
| TS-002 | Hallazgos incluyen informacion financiera y noticias del sector | Se genera el analisis | Oportunidades estan priorizadas por horizonte (corto/mediano/largo), cada una con rationale | Alta |
| TS-003 | Stakeholders identificados incluyen un CTO y un CFO | Se genera el analisis | CTO tiene dolores de tecnologia/legacy, CFO tiene dolores de eficiencia/costos, ambos con approach diferenciado | Alta |
| TS-004 | Hallazgos revelan que el cliente opera en Brasil ademas de Colombia | Se genera el analisis | internationalAlert.isInternational=true, languages incluye "PT", recommendation sugiere materiales en portugues | Media |
| TS-005 | El comercial marca 3 hallazgos como irrelevantes | Se actualizan los findings | Los 3 hallazgos tienen isRelevant=false, las notas del comercial se persisten | Media |
| TS-006 | El RAG tiene 2 casos de exito del sector seguros | Se genera el analisis para una aseguradora | Las oportunidades incluyen matchedCaseStudies con los 2 casos relevantes | Alta |
| TS-007 | La investigacion tiene status DEEP_SEARCHING (no completada) | Se intenta generar el analisis | Error INVESTIGATION_NOT_COMPLETED | Alta |
| TS-008 | El motor de IA no responde (timeout) | Se intenta generar el analisis | Error AI_ANALYSIS_FAILED, no se persiste analisis parcial | Alta |

---

## Referencias

- SPEC-L1-account-planning v2.0.0 — Dominio
- SPEC-L2-execute-research v1.0.0 — Investigacion que genera los hallazgos
- context/scope.md — Seccion "Dashboard de resultados"
- context/Promejemplo.txt — Estructura de analisis esperada por el equipo comercial

---

## Especificaciones relacionadas

| Spec ID | Nivel | Version | Relacion |
|---------|-------|---------|----------|
| SPEC-L1-account-planning | L1 | 2.0.0 | Parent — dominio |
| SPEC-L2-execute-research | L2 | 1.0.0 | Upstream — provee los hallazgos |
| SPEC-L2-generate-portfolio | L2 | 1.0.0 | Downstream — usa el analisis para generar el portfolio |

---

## Historial de cambios

| Version | Fecha | Autor | Descripcion del cambio |
|---------|-------|-------|----------------------|
| 1.0.0 | 2026-04-09 | Juan Gabriel Colorado | Version inicial: analisis IA con dashboard de 9 secciones |
