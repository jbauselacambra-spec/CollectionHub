# Conceptual Overview

Version: 1.0

Status: APPROVED

---

# Objetivo

El Dominio Conceptual define el conocimiento del negocio sobre el que se construye CollectionHub.

No describe clases, bases de datos, APIs ni detalles técnicos.

Su propósito es responder a una única pregunta:

> **¿Cómo funciona el coleccionismo estratégico desde el punto de vista del dominio?**

Todos los conceptos definidos en esta carpeta son independientes de cualquier tecnología y representan el lenguaje común del proyecto.

---

# Visión General

CollectionHub no se construye alrededor de objetos.

Se construye alrededor de decisiones.

El dominio describe cómo un Collector transforma información en decisiones estratégicas que incrementan su Strategic Capital y reducen su Strategic Debt.

Todo el modelo conceptual gira alrededor de ese objetivo.

---

# Flujo Conceptual

```
                    Collector
                        │
                        ▼
                  Portfolio Strategy
                        │
                        ▼
                   Portfolio
                        │
        ┌───────────────┼───────────────┐
        ▼               ▼               ▼
   Collections      Budget        Objectives
        │
        ▼
   Collectibles
        │
        ▼
  Market Information
        │
        ▼
 Strategic Economics
        │
        ▼
  Decision Model
        │
        ▼
 Strategic Reasoning
        │
        ▼
 Recommendation Model
        │
        ▼
 Strategic Capital
        │
        ▼
      Collector
```

El proceso forma un ciclo continuo.

Cada decisión modifica el estado del Portfolio y genera nuevo conocimiento para futuras decisiones.

---

# Relación entre los documentos

## 01_DOMAIN_UNIVERSE

Define el universo del dominio.

Responde a la pregunta:

> ¿Qué existe dentro de CollectionHub?

---

## 02_UBIQUITOUS_LANGUAGE

Define el lenguaje común utilizado por el proyecto.

Garantiza que todos los conceptos tengan un único significado.

---

## 03_STRATEGIC_ECONOMICS

Describe cómo se genera, conserva y utiliza el valor estratégico dentro del Portfolio.

Introduce conceptos como:

- Opportunity Cost
- Strategic Capital
- Strategic Debt
- Liquidity
- Scarcity

---

## 04_DECISION_MODEL

Describe el proceso general mediante el cual el sistema toma decisiones.

No explica el razonamiento interno.

Solo define el flujo de decisión.

---

## 05_STRATEGIC_REASONING

Describe cómo razona CollectionHub.

Aplica los principios definidos por la Constitución y los Core Principles para evaluar cada situación.

Es el núcleo intelectual del sistema.

---

## 06_RECOMMENDATION_MODEL

Define cómo se comunican las decisiones al Collector.

Las recomendaciones deben ser:

- explicables
- justificadas
- trazables
- alineadas con la Constitución

---

## 07_STRATEGIC_CAPITAL

Define el principal indicador estratégico del sistema.

No mide únicamente dinero.

Representa la capacidad futura del Collector para tomar buenas decisiones.

---

## 08_PORTFOLIO_STRATEGY

Describe cómo se organiza el Portfolio.

Define prioridades, asignación de recursos y reglas estratégicas entre colecciones.

---

# Dependencias Conceptuales

Los documentos siguen una relación de dependencia lógica.

```
Domain Universe
        │
        ▼
Ubiquitous Language
        │
        ▼
Strategic Economics
        │
        ▼
Decision Model
        │
        ▼
Strategic Reasoning
        │
        ▼
Recommendation Model
        │
        ▼
Portfolio Strategy
        │
        ▼
Strategic Capital
```

Cada documento amplía al anterior.

Ningún documento sustituye a otro.

---

# Principios del Dominio Conceptual

El Dominio Conceptual debe cumplir siempre las siguientes reglas:

- Es independiente de la tecnología.
- Es independiente de la implementación.
- Es independiente de la interfaz de usuario.
- Es independiente de la base de datos.
- Es independiente de la Inteligencia Artificial utilizada.
- Debe poder entenderse sin escribir una sola línea de código.

---

# Qué NO pertenece al Dominio Conceptual

Los siguientes elementos forman parte del Dominio Técnico:

- Entidades
- Value Objects
- Aggregates
- Domain Services
- Domain Events
- Repositories
- Specifications
- Policies

El Dominio Conceptual únicamente define el conocimiento del negocio.

---

# Relación con el Dominio Técnico

El Dominio Técnico implementa el conocimiento definido por el Dominio Conceptual.

Nunca debe modificarlo.

La relación entre ambos es la siguiente:

```
FOUNDATION
        │
        ▼
DOMAIN CONCEPTUAL
        │
        ▼
DOMAIN TECHNICAL
        │
        ▼
ARCHITECTURE
        │
        ▼
IMPLEMENTATION
```

Toda implementación debe respetar el modelo conceptual.

---

# Regla de Oro

Si una implementación técnica contradice el Dominio Conceptual, la implementación es incorrecta.

El modelo conceptual constituye la fuente de verdad del dominio.

---

# Objetivo Final

Construir un modelo de conocimiento estable, coherente y compartido que permita evolucionar CollectionHub durante años sin perder consistencia conceptual.

El éxito del proyecto dependerá de que todas las decisiones técnicas puedan explicarse utilizando únicamente los conceptos definidos en este Dominio Conceptual.