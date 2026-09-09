# Technical Overview

Version: 1.0

Status: APPROVED

---

# Objetivo

El Dominio Técnico representa la implementación conceptual del negocio.

No define tecnologías concretas ni decisiones de infraestructura.

Su propósito es transformar el conocimiento definido en el Dominio Conceptual en un modelo técnico coherente, mantenible y preparado para ser implementado.

Responde a una única pregunta:

> **¿Cómo debe modelarse el dominio para preservar el conocimiento del negocio durante la implementación?**

---

# Principios del Dominio Técnico

El Dominio Técnico se rige por los siguientes principios:

- El código implementa el dominio, nunca lo redefine.
- Toda decisión técnica debe poder justificarse desde el Dominio Conceptual.
- Las entidades representan identidad y comportamiento, no estructuras de datos.
- Los Value Objects representan conceptos del dominio sin identidad.
- Los Aggregates protegen la consistencia del negocio.
- Los Domain Services encapsulan comportamiento que no pertenece a una entidad.
- Los Domain Events representan hechos ocurridos en el dominio.
- Las Policies contienen reglas complejas del negocio.
- Las Specifications permiten expresar reglas reutilizables.
- Los Repositories representan contratos de persistencia, nunca implementaciones.

---

# Organización

El Dominio Técnico se divide en las siguientes áreas.

```
01_TECHNICAL
│
├── 00_TECHNICAL_OVERVIEW.md
│
├── 01_ENTITIES
│
├── 02_VALUE_OBJECTS
│
├── 03_AGGREGATES
│
├── 04_DOMAIN_SERVICES
│
├── 05_DOMAIN_EVENTS
│
├── 06_POLICIES
│
├── 07_SPECIFICATIONS
│
└── 08_REPOSITORIES
```

Cada carpeta representa una responsabilidad concreta dentro del dominio.

---

# Flujo de Construcción

El Dominio Técnico debe evolucionar siguiendo siempre el mismo orden.

```
Conceptual Domain
        │
        ▼
Entities
        │
        ▼
Value Objects
        │
        ▼
Aggregates
        │
        ▼
Policies
        │
        ▼
Domain Services
        │
        ▼
Domain Events
        │
        ▼
Specifications
        │
        ▼
Repositories
```

Cada nivel depende del anterior.

Nunca debe construirse en sentido inverso.

---

# Relación con el Dominio Conceptual

El Dominio Conceptual responde a:

> ¿Qué significa cada concepto?

El Dominio Técnico responde a:

> ¿Cómo representamos esos conceptos sin perder su significado?

Ejemplo:

```
Conceptual

Portfolio representa la estrategia global del Collector.

        │

        ▼

Technical

Portfolio Aggregate
```

Otro ejemplo:

```
Conceptual

Strategic Capital

        │

        ▼

Technical

StrategicCapital Value Object
```

El modelo técnico nunca inventa conceptos nuevos.

Únicamente implementa los ya definidos.

---

# Capas del Modelo Técnico

## Entities

Representan objetos con identidad propia y ciclo de vida.

Ejemplos:

- Portfolio
- Collection
- Collectible
- Purchase
- Sale
- Recommendation
- Goal
- Opportunity

---

## Value Objects

Representan conceptos inmutables sin identidad.

Ejemplos:

- Money
- TargetPrice
- CompletionPercentage
- Liquidity
- StrategicCapital
- StrategicDebt
- Priority
- Condition

---

## Aggregates

Agrupan entidades relacionadas y garantizan la consistencia del dominio.

Cada Aggregate define:

- Root
- Boundary
- Invariants
- Consistency Rules

---

## Domain Services

Contienen comportamiento que no pertenece naturalmente a ninguna entidad.

Ejemplos futuros:

- Recommendation Service
- Pricing Service
- Portfolio Analysis Service
- Budget Allocation Service

---

## Domain Events

Representan hechos que ya han ocurrido.

Ejemplos:

- PurchaseRegistered
- SaleCompleted
- RecommendationGenerated
- CollectionCompleted
- TargetPriceReached

Los eventos describen hechos.

Nunca órdenes.

---

## Policies

Agrupan reglas complejas del negocio.

Ejemplos:

- Purchase Policy
- Budget Policy
- Collection Policy
- Portfolio Policy

---

## Specifications

Representan condiciones reutilizables del dominio.

Ejemplos:

- CanPurchase
- IsStrategicOpportunity
- CanCompleteCollection
- HasAvailableBudget

---

## Repositories

Representan contratos para acceder al estado persistente del dominio.

No contienen lógica de negocio.

No conocen la base de datos.

No conocen Entity Framework.

---

# Dependencias

Las dependencias siempre deben apuntar hacia el dominio.

```
Foundation
        │
        ▼
Conceptual Domain
        │
        ▼
Technical Domain
        │
        ▼
Application
        │
        ▼
Infrastructure
```

Nunca debe existir una dependencia inversa.

---

# Principios Arquitectónicos

El Dominio Técnico debe ser:

- Independiente de Entity Framework.
- Independiente de SQL Server.
- Independiente de ASP.NET.
- Independiente de IA.
- Independiente de APIs externas.
- Independiente del Frontend.

El dominio debe poder ejecutarse incluso sin infraestructura.

---

# Document First Development

CollectionHub adopta la filosofía **Document First Development (DFD)**.

Antes de implementar cualquier componente del dominio debe existir su documentación correspondiente.

Esto aplica a:

- Entidades
- Value Objects
- Aggregates
- Policies
- Domain Services
- Domain Events
- Specifications
- Repositories

La documentación constituye la fuente de verdad del modelo técnico.

El código únicamente implementa dicho modelo.

---

# Relación entre las piezas

```
                 Portfolio
                      │
        ┌─────────────┼─────────────┐
        ▼             ▼             ▼
 Collection      Recommendation    Budget
        │
        ▼
 Collectible
        │
        ▼
 Opportunity
        │
        ▼
 Purchase
        │
        ▼
 Domain Event
        │
        ▼
 Strategic Capital
```

Este diagrama representa una visión simplificada del comportamiento del dominio.

Será refinado conforme evolucione el modelo.

---

# Regla de Oro

Si una implementación técnica contradice el Dominio Conceptual o la Foundation, la implementación es incorrecta.

El dominio siempre prevalece sobre el código.

---

# Objetivo Final

Construir un modelo técnico estable, expresivo y alineado con el negocio, de forma que la implementación sea una consecuencia natural del diseño y no el lugar donde se toman las decisiones de dominio.