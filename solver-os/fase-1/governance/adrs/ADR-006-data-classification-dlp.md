# ADR-006: Clasificacion de datos y prevencion de fuga (DLP)

| Campo | Valor |
|-------|-------|
| **ADR ID** | ADR-006 |
| **Estado** | ACCEPTED |
| **Fecha** | 2026-04-09 |
| **Autor** | Equipo de Arquitectura |
| **PR** | Pendiente |
| **Reemplaza** | N/A |
| **Reemplazado por** | N/A |

## Contexto

La generacion de codigo asistida por IA introduce riesgos especificos de fuga de datos:

1. **PII en prompts**: al pasar contexto a la IA, se pueden incluir datos personales reales (emails, documentos, datos financieros)
2. **PII en codigo generado**: la IA puede generar logs, respuestas API o fixtures de test que contengan datos sensibles
3. **Datos sensibles en respuestas LLM**: las respuestas de modelos pueden incluir o inferir datos personales del contexto
4. **Ausencia de clasificacion**: sin una taxonomia de datos, no hay criterio objetivo para decidir que proteger

Adicionalmente, la normativa colombiana (Ley 1581 de 2012) y el GDPR exigen controles tecnicos para la proteccion de datos personales, incluyendo Privacy by Design y Privacy by Default.

## Decision

Todo sistema desarrollado bajo el framework debe implementar:

### 1. Clasificacion de datos obligatoria

Toda entidad definida en una especificacion L1 debe clasificar sus atributos segun esta taxonomia:

| Nivel | Descripcion | Ejemplos | Tratamiento |
|-------|------------|----------|-------------|
| **Publico** | Informacion de libre acceso | Nombre del producto, precios publicados | Sin restriccion |
| **Interno** | Informacion operativa no sensible | IDs internos, timestamps, estados | No exponer externamente |
| **Confidencial** | Informacion de negocio protegida | Contratos, estrategias, metricas financieras | Acceso restringido, cifrado en reposo |
| **Sensible (PII)** | Datos personales identificables | Email, documento de identidad, telefono, direccion, datos de salud, datos financieros personales | Cifrado, masking en logs, anonimizacion en ambientes no productivos, consentimiento requerido |

### 2. Principios DLP transversales

- **Privacy by Design**: la proteccion de datos se disena desde la especificacion, no se agrega despues
- **Privacy by Default**: el comportamiento por defecto es el mas restrictivo (no loggear PII, no exponer datos sensibles)
- **Minimo dato necesario**: solo recolectar y procesar los datos estrictamente necesarios para el caso de uso
- **DLP en el ciclo de IA**: los prompts y respuestas de LLM deben pasar por filtros DLP antes y despues de la invocacion

### 3. Requisitos en especificaciones

- **Spec L1**: cada entidad debe tener sus atributos clasificados segun la taxonomia
- **Spec L2**: debe incluir seccion "Clasificacion de datos" indicando que campos son PII, como se protegen en logs, respuestas y persistencia
- **Spec L3**: si un cambio agrega o modifica campos, debe actualizar la clasificacion

## Alternativas evaluadas

| Opcion | Descripcion | Pros | Contras |
|--------|------------|------|---------|
| **Clasificacion en spec + DLP transversal** (elegida) | Clasificar datos en la spec y aplicar DLP como regla de gobernanza | Trazable, controlado desde la especificacion, cumple normativa | Requiere disciplina en cada spec |
| **DLP solo en infraestructura** | Aplicar filtros DLP a nivel de red/WAF sin control en codigo | Transparente para devs | No previene PII en logs internos, no controla prompts a LLM, falso sentido de seguridad |
| **Revision manual post-generacion** | Revisar manualmente si hay PII despues de generar | Sin overhead en specs | No escala, depende del criterio individual, alto riesgo de omision |

## Consecuencias

### Positivas
- Toda PII es identificable y trazable desde la especificacion
- Los quality gates pueden validar automaticamente que PII no se exponga en logs o respuestas
- Cumplimiento con Ley 1581 de 2012 y GDPR por diseno
- Reduce el riesgo de fuga de datos en generacion con IA
- Los prompts a LLM se sanitizan antes de enviar

### Negativas
- Agrega un paso obligatorio en la creacion de specs (clasificar atributos)
- Requiere que el equipo conozca la taxonomia de clasificacion
- Puede generar friccion inicial hasta que se adopte como habito

## Cumplimiento

- En especificaciones L1: seccion "Clasificacion de datos" obligatoria por entidad
- En especificaciones L2: seccion "Proteccion de datos" con tratamiento por campo PII
- En Gate 2: validar que logs y respuestas API no expongan campos clasificados como Sensible
- En Gate 3: el reviewer verifica clasificacion correcta y tratamiento adecuado
- En prompts: nunca incluir datos reales clasificados como Sensible o Confidencial (usar datos sinteticos)

## Normativas de referencia

| Normativa | Ambito | Requisitos clave |
|-----------|--------|-----------------|
| **Ley 1581 de 2012** | Colombia | Consentimiento, finalidad, acceso y circulacion restringida, seguridad |
| **GDPR** | Union Europea | Privacy by Design, Privacy by Default, minimizacion de datos, derecho al olvido |
| **ISO 27001** | Internacional | Clasificacion de activos de informacion, controles de acceso |

## ADRs relacionados

- ADR-001: Ports and Adapters — los filtros DLP se implementan como adaptadores, no en el dominio
- ADR-003: Traceability — la clasificacion de datos es parte de la trazabilidad del sistema
- ADR-004: Dependency Management — las herramientas DLP deben estar autorizadas en la spec
