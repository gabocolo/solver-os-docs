# Oportunidades de agentizacion

Framework para identificar, evaluar y priorizar procesos candidatos a ser automatizados mediante agentes de IA.

---

## Como identificar oportunidades

Buscar procesos que cumplan al menos 3 de estos criterios:
1. **Repetitivo**: se ejecuta frecuentemente (diario, semanal)
2. **Predecible**: tiene pasos claros y definidos
3. **Alto volumen**: consume tiempo significativo del equipo
4. **Cross-team**: aplica a multiples equipos o clientes
5. **Basado en reglas**: sigue logica deterministica, no juicio subjetivo

---

## Matriz de evaluacion

Completar una fila por cada proceso candidato:

| # | Proceso | Frecuencia | Esfuerzo actual (hrs/semana) | Repetitividad (1-5) | Aplicabilidad cross-team (1-5) | Potencial de automatizacion (1-5) | Score total | Prioridad |
|---|---------|-----------|------------------------------|---------------------|-------------------------------|----------------------------------|------------|-----------|
| 1 | {{proceso 1}} | {{diario/semanal/mensual}} | {{horas}} | {{1-5}} | {{1-5}} | {{1-5}} | {{suma}} | {{Alta/Media/Baja}} |
| 2 | {{proceso 2}} | {{frecuencia}} | {{horas}} | {{1-5}} | {{1-5}} | {{1-5}} | {{suma}} | {{prioridad}} |
| 3 | {{proceso 3}} | {{frecuencia}} | {{horas}} | {{1-5}} | {{1-5}} | {{1-5}} | {{suma}} | {{prioridad}} |
| 4 | {{proceso 4}} | {{frecuencia}} | {{horas}} | {{1-5}} | {{1-5}} | {{1-5}} | {{suma}} | {{prioridad}} |
| 5 | {{proceso 5}} | {{frecuencia}} | {{horas}} | {{1-5}} | {{1-5}} | {{1-5}} | {{suma}} | {{prioridad}} |

**Scoring:**
- 12-15: Prioridad **Alta** — candidato inmediato
- 8-11: Prioridad **Media** — evaluar en siguiente ciclo
- 3-7: Prioridad **Baja** — mantener en backlog

---

## Categorias de oportunidades

### Oportunidades internas (procesos propios)

| Area | Procesos candidatos tipicos |
|------|---------------------------|
| Desarrollo | Generacion de boilerplate, scaffolding de proyectos, creacion de specs desde historias de usuario |
| QA | Generacion de tests desde specs, analisis de cobertura, reportes de calidad |
| DevOps | Configuracion de pipelines, generacion de IaC, monitoreo de deployments |
| Documentacion | Generacion de READMEs, documentacion de APIs, changelogs |
| Code review | Pre-revision automatizada, deteccion de anti-patrones, verificacion de estandares |

### Oportunidades externas (procesos de clientes)

| Industria | Procesos candidatos tipicos |
|-----------|---------------------------|
| Financiero | Generacion de reportes regulatorios, validacion de transacciones, reconciliacion |
| Retail | Gestion de catalogos, generacion de descripciones, analisis de inventario |
| Salud | Procesamiento de formularios, generacion de reportes clinicos |
| Logistica | Optimizacion de rutas, tracking automatizado, generacion de manifiestos |
| General | Onboarding de sistemas, migracion de datos, integracion de APIs |

---

## ROI estimado

Para cada oportunidad priorizada, estimar:

| Metrica | Valor |
|---------|-------|
| **Esfuerzo actual** | {{horas/semana que consume hoy}} |
| **Esfuerzo con agente** | {{horas/semana estimadas con automatizacion}} |
| **Ahorro semanal** | {{diferencia}} |
| **Costo de desarrollo del agente** | {{horas estimadas de desarrollo}} |
| **Tiempo de recuperacion** | {{semanas para recuperar la inversion}} |

---

## Siguiente paso

Para cada oportunidad de alta prioridad, crear un diseno de acelerador usando `ACCELERATOR-DESIGN-V1.md`.
