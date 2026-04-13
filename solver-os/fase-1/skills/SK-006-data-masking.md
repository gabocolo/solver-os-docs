# Definicion de Skill

| Campo | Valor |
|-------|-------|
| **ID** | SK-006 |
| **Nombre** | Data Masking en codigo generado |
| **Tipo** | Build |
| **Version** | 1.0.0 |
| **Owner** | Lider tecnico |
| **Fase del framework** | Fase 3 — Generacion controlada |
| **Estado** | ACTIVE |

---

## Descripcion

Genera la implementacion de filtros de data masking y anonimizacion para campos clasificados como Sensible (PII) en la especificacion. Produce el codigo necesario para enmascarar datos en logs, respuestas API y ambientes no productivos, siguiendo los patrones definidos en ADR-006 y los estandares de observabilidad.

---

## Cuando usar

- Cuando una spec L2 tiene campos clasificados como Sensible (PII)
- Cuando se implementa un caso de uso que maneja datos personales (email, documento, telefono, datos financieros)
- Cuando se necesita preparar datos para ambientes no productivos (anonimizacion de dumps)

---

## Cuando NO usar

- Para campos clasificados como Publico o Interno (no requieren masking)
- Para cifrado de datos en reposo (eso es configuracion de infraestructura, no un skill de generacion)
- Para validacion de input de usuario (eso es parte del caso de uso, no un skill separado)

---

## Input requerido

| Input | Descripcion | Obligatorio |
|-------|------------|-------------|
| Especificacion L2 con clasificacion de datos | Spec que identifique campos PII y su tratamiento | Si |
| ADR-006 | Decision de clasificacion y DLP | Si |
| Estandares de observabilidad | Patrones de masking definidos en OBSERVABILITY-STANDARDS.md | Si |
| Stack tecnologico | Lenguaje y framework del proyecto | Si |

---

## Output esperado

1. **Filtro de PII para logging**: componente/middleware que intercepta logs y aplica masking automatico a campos sensibles antes de persistir
2. **Serializador seguro para API responses**: logica que enmascara campos PII en respuestas API segun el nivel de autorizacion del consumidor
3. **Tests GWT** para el filtro:
   - Given un log con email real → When se persiste → Then el email esta enmascarado
   - Given una respuesta API con documento → When se serializa → Then el documento muestra solo ultimos 4 digitos
   - Given un campo Publico en el log → When se persiste → Then el campo no se modifica

---

## Ejemplo de invocacion

```
Invoca el skill Data Masking con:
- Contexto del cambio: Implementar filtro PII para el modulo de clientes
- Especificacion: manage-clients v1.0.0
- Campos PII segun spec: email (Sensible), documento (Sensible), nombre (Interno)
- Stack: {{lenguaje/framework}}
- Patrones de masking: segun OBSERVABILITY-STANDARDS.md

IMPORTANTE: Generar SOLO el filtro de masking y sus tests.
No modificar la logica de dominio. El filtro opera en la capa de infraestructura (adaptador).
Usar datos sinteticos en los tests, nunca datos reales.
```

---

## Metricas

| Metrica | Valor actual | Objetivo |
|---------|-------------|---------|
| % retrabajo | Por medir | < 10% |
| PII detectada en logs post-implementacion | Por medir | 0 |
| Cobertura de campos PII de la spec | Por medir | 100% |

---

## Historial de cambios

| Version | Fecha | Autor | Descripcion |
|---------|-------|-------|------------|
| 1.0.0 | 2026-04-09 | Equipo de Arquitectura | Creacion inicial del skill |
