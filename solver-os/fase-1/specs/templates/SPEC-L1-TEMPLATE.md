# Especificacion de Dominio (Nivel 1)

| Campo | Valor |
|-------|-------|
| **Spec ID** | SPEC-L1-{{domain-name}} |
| **Nivel** | L1 — Dominio |
| **Version** | {{X.Y.Z}} |
| **Estado** | `DRAFT` | `IN_REVIEW` | `APPROVED` | `IMPLEMENTED` | `DEPRECATED` |
| **Creado por** | @{{autor}} |
| **Fecha de creacion** | {{YYYY-MM-DD}} |
| **Aprobado por** | @{{aprobador}} |
| **Fecha de aprobacion** | {{YYYY-MM-DD}} |
| **PR** | {{link al PR}} |

---

## Descripcion general

_Resumen breve de que trata este dominio y cual es su proposito dentro del producto o sistema._

---

## Dominio

**Nombre:** {{nombre del dominio}}

### Limite del contexto (Bounded Context)

**Responsabilidad:** {{que es responsabilidad exclusiva de este contexto}}

**Fuera de alcance:** {{que NO es responsabilidad de este contexto, aunque parezca relacionado}}

| Contexto externo | Relacion | Descripcion |
|-----------------|----------|-------------|
| {{Contexto 1}} | {{Upstream / Downstream / ACL}} | {{como se relacionan}} |

> **Upstream:** ellos publican, nosotros consumimos. **Downstream:** nosotros publicamos, ellos consumen. **ACL (Anti-Corruption Layer):** traducimos sus datos a nuestro modelo.

---

## Objetivo de negocio

_Que problema resuelve este dominio? Que valor genera para el usuario o para la organizacion?_

{{Describe el objetivo de negocio que este dominio sirve.}}

---

## Contexto

_Informacion de fondo necesaria para entender el dominio: estado actual del sistema, decisiones previas, dependencias con otros dominios, supuestos clave._

---

## Modelo de dominio

### Agregados

_Cada agregado define un limite de consistencia. Las operaciones que modifican estado deben pasar por la raiz del agregado. Las entidades y value objects dentro de un agregado no se acceden directamente desde fuera._

| Agregado (raiz) | Entidades internas | Value Objects | Invariantes |
|-----------------|-------------------|---------------|-------------|
| {{Agregado 1}} | {{Entidad1, Entidad2}} | {{VO1, VO2}} | {{regla de consistencia del agregado}} |

### Detalle de entidades

| Entidad | Tipo | Agregado | Descripcion | Atributos clave |
|---------|------|----------|------------|----------------|
| {{Entidad 1}} | Raiz de agregado | {{Agregado}} | {{descripcion}} | {{atributos}} |
| {{Entidad 2}} | Entidad interna | {{Agregado}} | {{descripcion}} | {{atributos}} |
| {{Entidad 3}} | Value Object | {{Agregado}} | {{descripcion}} | {{atributos}} |

> **Tipo:** _Raiz de agregado_ = tiene identidad unica y ciclo de vida independiente. _Entidad interna_ = existe solo dentro de su agregado. _Value Object_ = sin identidad, definido por sus atributos, inmutable.

---

## Reglas de negocio

| ID | Regla | Descripcion | Justificacion |
|----|-------|------------|---------------|
| BR-001 | {{nombre de la regla}} | {{descripcion detallada}} | {{por que existe esta regla}} |
| BR-002 | {{nombre de la regla}} | {{descripcion detallada}} | {{por que existe esta regla}} |
| BR-003 | {{nombre de la regla}} | {{descripcion detallada}} | {{por que existe esta regla}} |

---

## Eventos de dominio

_Hechos relevantes que ocurren dentro de este dominio y que otros contextos pueden necesitar conocer. Se definen aqui a nivel de negocio y se especifican tecnicamente en L2._

| Evento | Disparado cuando | Datos relevantes | Consumidores potenciales |
|--------|-----------------|-----------------|------------------------|
| {{EventoCreado}} | {{condicion que lo dispara}} | {{datos clave del evento}} | {{contextos o sistemas interesados}} |

---

## Politicas de negocio

{{Restricciones organizacionales o de negocio que aplican a este dominio.}}

- {{Politica 1}}
- {{Politica 2}}

---

## Validaciones

{{Que debe ser verdadero para que las operaciones de este dominio procedan.}}

| Validacion | Condicion | Error si falla |
|-----------|----------|---------------|
| {{Validacion 1}} | {{condicion}} | {{error}} |
| {{Validacion 2}} | {{condicion}} | {{error}} |

---

## Casos de uso

_Describe los casos de uso principales de este dominio desde la perspectiva del usuario._

### {{Caso de uso 1: Nombre}}

```
Como [tipo de usuario],
quiero [accion],
para que [beneficio].
```

**Comportamiento esperado:**

1. {{El usuario hace X...}}
2. {{El sistema responde con Y...}}
3. {{...}}

### {{Caso de uso 2: Nombre}}

```
Como [tipo de usuario],
quiero [accion],
para que [beneficio].
```

**Comportamiento esperado:**

1. {{Paso 1...}}
2. {{Paso 2...}}

---

## Criterios de aceptacion del dominio

- [ ] {{Criterio 1: descripcion verificable}}
- [ ] {{Criterio 2: descripcion verificable}}
- [ ] {{Criterio 3: descripcion verificable}}

---

## Restricciones y consideraciones

| Tipo | Descripcion |
|------|------------|
| **Limitaciones de negocio** | {{restricciones del negocio que aplican}} |
| **Integraciones necesarias** | {{sistemas externos o dominios con los que se debe integrar}} |
| **Consideraciones de seguridad** | {{requisitos de seguridad especificos del dominio}} |
| **Consideraciones de rendimiento** | {{expectativas de rendimiento o volumetria}} |

---

## Lenguaje ubicuo del dominio (Glosario)

_Estos terminos son obligatorios en codigo, specs, tests y conversaciones. La columna "Nombre en codigo" define como debe aparecer en la implementacion._

| Termino | Definicion | Nombre en codigo |
|---------|-----------|-----------------|
| {{Termino 1}} | {{definicion}} | {{TerminoEnCodigo}} |
| {{Termino 2}} | {{definicion}} | {{TerminoEnCodigo}} |

---

## Diseno / Mockups

_Incluye enlaces a disenos, wireframes o mockups si aplica. Si no hay diseno formal, describe la interfaz esperada._

---

## Referencias

- {{Enlace o referencia a documentacion de negocio}}
- {{Enlace a ADRs relevantes}}
- {{Enlace a investigaciones o decisiones previas}}

---

## Especificaciones relacionadas

| Spec ID | Nivel | Version | Descripcion |
|---------|-------|---------|------------|
| {{SPEC-L2-xxx}} | L2 | {{X.Y.Z}} | {{descripcion}} |

---

## Historial de cambios

| Version | Fecha | Autor | Descripcion del cambio |
|---------|-------|-------|----------------------|
| {{X.Y.Z}} | {{YYYY-MM-DD}} | {{autor}} | {{descripcion}} |
