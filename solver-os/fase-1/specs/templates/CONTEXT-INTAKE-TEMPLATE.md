# Captura de Contexto Inicial

| Campo | Valor |
|-------|-------|
| **Proyecto** | {{nombre del proyecto}} |
| **Version** | {{1.0.0}} |
| **Fecha** | {{YYYY-MM-DD}} |
| **Autor** | {{nombre del autor}} |
| **Estado** | DRAFT / REVIEWED / APPROVED |

---

## 1. Objetivo de negocio

_Que problema resuelve este proyecto? Que valor genera?_

{{Describe en 2-3 parrafos el objetivo de negocio. Se especifico: que dolor del usuario o del negocio se ataca, que resultado medible se espera, y por que es prioritario ahora.}}

---

## 2. Usuarios y roles

| Rol | Descripcion | Permisos clave |
|-----|------------|---------------|
| {{Rol 1}} | {{descripcion del rol}} | {{que puede hacer}} |
| {{Rol 2}} | {{descripcion del rol}} | {{que puede hacer}} |
| {{Rol 3}} | {{descripcion del rol}} | {{que puede hacer}} |

---

## 3. Flujos principales

_Describe los flujos de negocio en Mermaid (optimiza tokens, mejor que ASCII). Minimo el flujo principal (happy path) y un flujo de error._

### Flujo principal (happy path)

```mermaid
flowchart TB
    A[{{Paso inicial}}] --> B[{{Paso 2}}]
    B --> C{{{Decision}}}
    C -->|Si| D[{{Paso exitoso}}]
    C -->|No| E[{{Paso alternativo}}]
    D --> F[{{Resultado final}}]
```

### Flujo de error

```mermaid
flowchart TB
    A[{{Paso inicial}}] --> B[{{Paso que falla}}]
    B --> C[{{Manejo de error}}]
    C --> D[{{Respuesta al usuario}}]
```

---

## 4. Entidades y datos

| Entidad | Descripcion | Datos clave | Clasificacion PII |
|---------|------------|-------------|-------------------|
| {{Entidad 1}} | {{descripcion}} | {{campos principales}} | {{Publico/Interno/Confidencial/Sensible}} |
| {{Entidad 2}} | {{descripcion}} | {{campos principales}} | {{Publico/Interno/Confidencial/Sensible}} |
| {{Entidad 3}} | {{descripcion}} | {{campos principales}} | {{Publico/Interno/Confidencial/Sensible}} |

---

## 5. Restricciones tecnicas

| Restriccion | Descripcion | Justificacion |
|------------|------------|---------------|
| {{Stack backend}} | {{.NET 10 / Java / etc}} | {{por que se eligio}} |
| {{Stack frontend}} | {{Angular / React / etc}} | {{por que se eligio}} |
| {{Base de datos}} | {{PostgreSQL / SQL Server / etc}} | {{por que se eligio}} |
| {{Cloud}} | {{Azure / AWS / on-prem}} | {{por que se eligio}} |

---

## 6. Integraciones externas

| Sistema externo | Tipo de integracion | Contrato conocido | Notas |
|----------------|--------------------|--------------------|-------|
| {{Sistema 1}} | {{API REST / Event / gRPC}} | {{Si/No — URL si existe}} | {{notas relevantes}} |
| {{Sistema 2}} | {{API REST / Event / gRPC}} | {{Si/No — URL si existe}} | {{notas relevantes}} |

---

## 7. Requisitos no funcionales

| Requisito | Valor esperado | Prioridad |
|-----------|---------------|-----------|
| Latencia | {{ms p95}} | Alta/Media/Baja |
| Concurrencia | {{usuarios simultaneos}} | Alta/Media/Baja |
| Disponibilidad | {{99.x%}} | Alta/Media/Baja |
| Observabilidad | {{logs/trazas/metricas}} | Alta/Media/Baja |

---

## 8. Supuestos y preguntas abiertas

### Supuestos (lo que asumimos como verdadero)

- {{Supuesto 1}}
- {{Supuesto 2}}
- {{Supuesto 3}}

### Preguntas abiertas (requieren respuesta del negocio)

- [ ] {{Pregunta 1}}
- [ ] {{Pregunta 2}}
- [ ] {{Pregunta 3}}

---

## 9. Fuentes de contexto

| Fuente | Tipo | Ubicacion |
|--------|------|-----------|
| {{Transcripcion reunion kickoff}} | VTT/Texto | {{path o URL}} |
| {{Diagrama de arquitectura}} | Mermaid/Imagen | {{path o URL}} |
| {{Documento de negocio}} | PDF/Markdown | {{path o URL}} |

> **IMPORTANTE:** Los diagramas deben estar en Mermaid, no en ASCII. Mermaid es mas eficiente en tokens para la IA (recomendacion de Osvaldo Arias).

---

## 10. Criterios de exito del proyecto

- [ ] {{Criterio 1: verificable y medible}}
- [ ] {{Criterio 2: verificable y medible}}
- [ ] {{Criterio 3: verificable y medible}}

---

_Este template se llena ANTES de crear las especificaciones L1/L2/L3. Sirve como entrada estructurada para el agente de especificaciones._
