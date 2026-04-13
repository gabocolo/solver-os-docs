# Supuestos de la solucion

Condiciones que se asumen como verdaderas para que esta solucion sea viable. Si alguno de estos supuestos no se cumple, se debe re-evaluar el impacto en la arquitectura, el alcance o el timeline.

---

## 1. Supuestos de negocio

| # | Supuesto | Impacto si NO se cumple |
|---|---------|------------------------|
| N-01 | **Tech&Solve tiene un portafolio de clientes activos suficiente** (minimo 5-10 cuentas) para justificar un sistema de account planning | Si hay pocas cuentas, un Excel o Notion es suficiente. El sistema solo se justifica con volumen |
| N-02 | **Los Account Managers tienen acceso a informacion de procesos del cliente** (pueden preguntar, observar, documentar procesos) | Sin informacion de procesos, el motor de IA no tiene input para analizar. El valor core se pierde |
| N-03 | **Existe voluntad comercial de pasar de un modelo reactivo a uno proactivo** — la direccion respalda que los equipos propongan sin que el cliente pida | Si la cultura sigue siendo reactiva, el sistema se convierte en un CRM glorificado |
| N-04 | **Los clientes estan abiertos a compartir informacion sobre sus procesos internos** a nivel general (no codigo, no datos sensibles) | Si los clientes no comparten, no hay input para el analisis de oportunidades |
| N-05 | **Las oportunidades identificadas por IA seran validadas por humanos** antes de presentarlas al cliente — no es un sistema autonomo | Si se presentan oportunidades sin validacion humana, se pierde credibilidad |
| N-06 | **Tech&Solve tiene o esta construyendo un catalogo de aceleradores/agentes** que se pueda cruzar con oportunidades | Sin catalogo, el matching es imposible. El catalogo puede empezar con 3-5 aceleradores minimo |
| N-07 | **El ciclo de venta de Tech&Solve permite propuestas proactivas** — no hay restricciones contractuales que impidan ofrecer servicios adicionales | Si los contratos limitan el scope, las oportunidades identificadas no se pueden ejecutar |

---

## 2. Supuestos tecnicos

| # | Supuesto | Impacto si NO se cumple |
|---|---------|------------------------|
| T-01 | **El equipo de desarrollo tiene experiencia con Angular y .NET** — no se requiere capacitacion significativa | Si no hay experiencia, el ramp-up puede agregar 2-4 semanas al MVP |
| T-02 | **Se tiene acceso a la API de Claude (Anthropic)** con un plan que soporte el volumen de analisis esperado | Sin acceso a Claude, el motor de IA no funciona. Alternativa: OpenAI GPT-4o, pero requiere adaptar prompts |
| T-03 | **El costo de tokens de Claude es asumible** — estimado: ~$50-150 USD/mes para 50-100 analisis mensuales (Sonnet + Haiku) | Si el costo es prohibitivo, se puede reducir frecuencia de analisis o usar solo Haiku |
| T-04 | **Se dispone de infraestructura Azure** (o AWS) para deployment | Si no hay cloud, se puede deployar on-premises con Docker, pero pierde auto-scaling y managed services |
| T-05 | **SQL Server esta disponible** como base de datos (licencia existente o Azure SQL) | Si no hay licencia, PostgreSQL es alternativa directa con minimo cambio en EF Core |
| T-06 | **La latencia de Claude API es aceptable** (2-5 segundos por analisis) para la experiencia de usuario | Si la latencia es mayor, se debe hacer el analisis completamente asincrono con notificacion |
| T-07 | **Redis esta disponible** para cache y queues (Azure Cache for Redis o instancia propia) | Sin Redis, se puede usar IMemoryCache (pierde cache distribuido) y SQL Server para Hangfire |
| T-08 | **El AI Engine en Python puede correr como contenedor separado** — el equipo de infra permite un servicio no-.NET | Si solo se permite .NET, hay que usar Semantic Kernel (Microsoft) como alternativa a LangChain |

---

## 3. Supuestos de datos

| # | Supuesto | Impacto si NO se cumple |
|---|---------|------------------------|
| D-01 | **No existe un CRM previo con datos de cuentas** que se deba migrar | Si existe (Salesforce, HubSpot, etc.), se necesita un plan de migracion o integracion que no esta en el alcance actual |
| D-02 | **La informacion de procesos del cliente se ingresa manualmente** por el Account Manager (no hay integracion automatica con sistemas del cliente) | Si se requiere integracion con sistemas del cliente, el alcance crece significativamente |
| D-03 | **Los datos de procesos del cliente no contienen PII ni datos regulados** — se trabaja a nivel de descripcion de procesos, no de datos transaccionales | Si hay PII, se necesita anonimizacion antes de enviar a Claude API y compliance (GDPR/Habeas Data) |
| D-04 | **El catalogo de aceleradores se mantiene manualmente** por el equipo de arquitectura (CRUD basico) | Si se requiere sincronizacion automatica con otro sistema, se necesita un adaptador adicional |
| D-05 | **Los usuarios del sistema son internos de Tech&Solve** — los clientes NO acceden al sistema | Si los clientes necesitan acceso, se requiere multi-tenancy, permisos granulares y una capa de presentacion diferente |
| D-06 | **El volumen inicial es bajo** — menos de 50 cuentas, menos de 100 analisis/mes | Si el volumen es alto desde el inicio, se debe escalar infra desde el dia 1 (no MVP) |

---

## 4. Supuestos de IA

| # | Supuesto | Impacto si NO se cumple |
|---|---------|------------------------|
| A-01 | **La calidad de las sugerencias de IA depende directamente de la calidad del input** — si el Account Manager ingresa descripciones vagas de procesos, las sugerencias seran vagas | Se necesita guia/training para los Account Managers sobre como documentar procesos del cliente |
| A-02 | **Claude API tiene disponibilidad suficiente** (SLA > 99%) para un uso no critico (no bloquea operacion si cae) | El sistema funciona sin IA (progressive enhancement), pero si cae frecuentemente, los usuarios pierden confianza |
| A-03 | **Los modelos de Claude no cambian de comportamiento drasticamente** entre versiones | Si Anthropic cambia un modelo, los prompts pueden necesitar ajuste. Mitigacion: versionar prompts, tener tests de regresion |
| A-04 | **El scoring de la IA es una sugerencia, no una verdad absoluta** — siempre requiere validacion humana | Si los equipos confian ciegamente en los scores, pueden proponer oportunidades irrelevantes (efecto "wow" del framework) |
| A-05 | **Los prompts no envian datos sensibles a Claude** — solo nombres de procesos, descripciones generales, industria | Si por error se envia PII, hay riesgo de compliance. Mitigacion: sanitizacion en el AI Engine antes de llamar a Claude |
| A-06 | **El auto-scoring periodico no genera costos excesivos** — re-evaluar 100 oportunidades semanalmente con Haiku cuesta ~$5-10 USD | Si el catalogo crece mucho, los costos de re-scoring pueden escalar. Mitigacion: solo re-evaluar oportunidades activas |

---

## 5. Supuestos de equipo y organizacion

| # | Supuesto | Impacto si NO se cumple |
|---|---------|------------------------|
| O-01 | **Hay un equipo dedicado de 2-3 personas** para construir el MVP en 4-6 sprints | Si el equipo es compartido con otros proyectos, el timeline se extiende proporcionalmente |
| O-02 | **Hay un Product Owner o stakeholder** que tome decisiones rapidas sobre priorizacion y alcance | Sin PO, las decisiones se retrasan y el MVP se extiende |
| O-03 | **Los Account Managers estan dispuestos a adoptar el sistema** y cambiar su flujo de trabajo | Si no adoptan, el sistema no genera valor. Se necesita change management |
| O-04 | **La direccion comercial respalda el piloto** y asigna 1-2 cuentas reales para probar | Sin cuentas reales, no hay forma de validar el valor del sistema |
| O-05 | **El equipo tiene acceso a ambientes de desarrollo y staging** desde el dia 1 | Si el aprovisionamiento de infra tarda semanas, el desarrollo se bloquea. Docker Compose mitiga parcialmente |

---

## 6. Supuestos de alcance (fuera de scope del MVP)

Estas funcionalidades se asumen **fuera del MVP** y se considerarian en fases posteriores:

| Funcionalidad | Por que esta fuera | Cuando entraria |
|---------------|-------------------|-----------------|
| Integracion con CRM (Salesforce, HubSpot) | Complejidad alta, cada CRM es diferente | Fase 2 si hay demanda |
| Acceso del cliente al sistema | Requiere multi-tenancy y permisos complejos | Fase 3 |
| Integracion con sistemas del cliente | Cada cliente es diferente, no escalable en MVP | Fase 2-3 segun demanda |
| App movil | El uso principal es desktop (Account Managers en oficina) | Solo si hay demanda real |
| Multi-idioma | Tech&Solve opera en espanol, clientes en LATAM | Fase 2 si hay clientes en ingles |
| Generacion automatica de propuestas comerciales (PDF) | Se puede hacer manual con los datos del sistema | Fase 2 |
| Integracion con facturacion / ERP | Fuera del dominio de account planning | No aplica |
| Analytics avanzados / BI | El dashboard basico es suficiente para MVP | Fase 2 con Power BI o similar |

---

## 7. Riesgos derivados de los supuestos

| Riesgo | Probabilidad | Impacto | Mitigacion |
|--------|-------------|---------|-----------|
| Account Managers no adoptan el sistema | Media | Alto | Involucrarlos desde el piloto, mostrar valor con SUS cuentas (workshop) |
| Calidad de input baja → sugerencias de IA malas | Alta | Alto | Template guiado en el wizard, ejemplos, validaciones, training |
| Claude API cambia precios significativamente | Baja | Medio | Abstraer via puerto (IAIEngine), poder cambiar a otro modelo |
| El catalogo de aceleradores no existe o es muy pobre | Media | Alto | Empezar con 3-5 aceleradores minimos, el sistema funciona sin matching perfecto |
| Los datos de procesos del cliente son confidenciales | Media | Alto | Sanitizacion en AI Engine, no enviar nombres de empresas ni datos especificos |
| El equipo no tiene experiencia con Python (AI Engine) | Baja | Bajo | El AI Engine es pequeno (~500 lineas), se puede hacer con un solo dev Python o con soporte de IA |

---

## Validacion de supuestos

Antes de iniciar el desarrollo, validar como minimo:

- [ ] **N-01**: Confirmar numero de cuentas activas que usarian el sistema
- [ ] **N-03**: Confirmar respaldo de direccion comercial para modelo proactivo
- [ ] **N-06**: Confirmar que existe catalogo de aceleradores (minimo 3-5)
- [ ] **T-02**: Confirmar acceso a Claude API y plan de costos
- [ ] **T-04**: Confirmar disponibilidad de infraestructura Azure/AWS
- [ ] **D-05**: Confirmar que solo usuarios internos de Tech&Solve acceden
- [ ] **O-01**: Confirmar equipo dedicado y disponibilidad
- [ ] **O-04**: Confirmar 1-2 cuentas para piloto

---

*Supuestos v1.0 — Account Planning Inteligente — Tech&Solve*
