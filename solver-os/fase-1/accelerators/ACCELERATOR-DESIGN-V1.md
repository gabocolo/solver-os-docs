# Diseno de acelerador transversal

| Campo | Valor |
|-------|-------|
| **Nombre** | {{nombre del acelerador}} |
| **Version** | {{X.Y.Z}} |
| **Owner** | {{responsable}} |
| **Fecha** | {{YYYY-MM-DD}} |
| **Estado** | [DRAFT | IN_REVIEW | APPROVED | ACTIVE | DEPRECATED] |

---

## Problema que resuelve

{{Que proceso repetitivo resuelve este acelerador? Que dolor elimina?}}

---

## Alcance

### Que hace
- {{funcionalidad 1}}
- {{funcionalidad 2}}
- {{funcionalidad 3}}

### Que NO hace
- {{exclusion 1}}
- {{exclusion 2}}

---

## Industrias aplicables

| Industria | Aplica | Notas de adaptacion |
|-----------|--------|-------------------|
| {{Industria 1}} | Si/No | {{que se personaliza}} |
| {{Industria 2}} | Si/No | {{que se personaliza}} |
| {{Industria 3}} | Si/No | {{que se personaliza}} |
| Universal | Si/No | {{si aplica a todas}} |

---

## Componentes reutilizables (base)

Estos componentes son **iguales** para cualquier cliente/dominio:

| Componente | Descripcion |
|-----------|------------|
| {{componente 1}} | {{que hace, por que es reutilizable}} |
| {{componente 2}} | {{que hace}} |
| {{componente 3}} | {{que hace}} |

---

## Puntos de personalizacion

Estos componentes **cambian** segun el cliente/dominio:

| Punto | Que se personaliza | Ejemplo |
|-------|-------------------|---------|
| {{punto 1}} | {{descripcion}} | {{ejemplo concreto}} |
| {{punto 2}} | {{descripcion}} | {{ejemplo}} |

---

## Arquitectura tecnica

{{Descripcion de alto nivel de la arquitectura. Incluir diagrama si es posible.}}

```
{{diagrama ASCII de la arquitectura}}
```

### Stack tecnologico

| Componente | Tecnologia |
|-----------|-----------|
| {{componente}} | {{tecnologia}} |

---

## Integraciones

| Sistema externo | Tipo de integracion | Proposito |
|----------------|-------------------|----------|
| {{sistema 1}} | {{API REST/Event/gRPC}} | {{para que}} |
| {{sistema 2}} | {{tipo}} | {{para que}} |

---

## Skills que compone

| Skill ID | Nombre | Rol en este acelerador |
|----------|--------|----------------------|
| {{SK-NNN}} | {{nombre}} | {{como se usa}} |

---

## Metricas de exito

| Metrica | Como se mide | Objetivo |
|---------|-------------|---------|
| Tiempo de setup por cliente | {{como}} | {{objetivo}} |
| % reutilizacion de componentes base | {{como}} | > 70% |
| Satisfaccion del cliente | {{como}} | {{objetivo}} |
| Tiempo de entrega vs sin acelerador | {{como}} | -{{X}}% |

---

## Plan de validacion

1. **Piloto interno:** {{con que equipo o proyecto se valida primero}}
2. **Piloto con cliente:** {{con que cliente se valida}}
3. **Feedback:** {{como se recoge y procesa el feedback}}
4. **Iteracion:** {{como se mejora basado en resultados}}

---

## Historial de cambios

| Version | Fecha | Autor | Descripcion |
|---------|-------|-------|------------|
| {{X.Y.Z}} | {{YYYY-MM-DD}} | {{autor}} | {{cambio}} |
