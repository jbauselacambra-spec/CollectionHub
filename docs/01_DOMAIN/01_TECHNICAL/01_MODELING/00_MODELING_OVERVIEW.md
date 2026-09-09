# Modeling Overview

Version: 1.0

Status: APPROVED

---

# Purpose

Este documento define la arquitectura global del Modelado Técnico de CollectionHub.

Su objetivo es establecer cómo se representa, organiza y relaciona el conocimiento del dominio antes de su traducción a código.

`00_MODELING_OVERVIEW.md` actúa como mapa maestro de `01_MODELING`.

No define conceptos concretos del dominio.

Define la estructura mediante la cual dichos conceptos serán modelados.

---

# Modeling Philosophy

CollectionHub no modela software.

CollectionHub modela conocimiento.

El código será una representación ejecutable de ese conocimiento.

Por tanto:

```text
Knowledge
    ↓
Domain Model
    ↓
Technical Representation
    ↓
Implementation

Position within the Architecture

El Modelado Técnico ocupa una posición intermedia entre el conocimiento conceptual y la implementación.

FOUNDATION
    ↓
¿Por qué existe CollectionHub?
    ↓
DOMAIN CONCEPTUAL
    ↓
¿Qué conceptos existen?
    ↓
TECHNICAL STANDARDS
    ↓
¿Cómo deben modelarse?
    ↓
DOMAIN MODELING
    ↓
¿Cómo se relacionan esos conceptos?
    ↓
IMPLEMENTATION
    ↓
¿Cómo ejecutamos el modelo?

Cada nivel depende conceptualmente del anterior.

Ningún nivel inferior deberá redefinir silenciosamente el significado establecido por un nivel superior.

Authority Hierarchy

Cuando exista una tensión entre decisiones documentales, deberá respetarse la siguiente jerarquía:

Project Constitution
        ↓
Core Principles
        ↓
Ubiquitous Language
        ↓
Conceptual Domain
        ↓
Technical Standards
        ↓
Domain Model
        ↓
Implementation

Una decisión de nivel inferior no podrá contradecir una decisión de nivel superior sin una revisión explícita de dicha decisión.

Modeling Categories

El modelo se organiza en ocho categorías fundamentales.

01_ENTITIES
02_VALUE_OBJECTS
03_AGGREGATES
04_DOMAIN_SERVICES
05_DOMAIN_EVENTS
06_POLICIES
07_SPECIFICATIONS
08_REPOSITORIES

Cada categoría representa una responsabilidad diferente.

Entities

Las Entities representan conceptos cuya identidad tiene significado dentro del dominio.

Su identidad permite distinguir una instancia de otra a lo largo del tiempo.

Las Entities contienen comportamiento y estado propios del concepto que representan.

Value Objects

Los Value Objects representan conceptos definidos por su valor y significado.

No poseen identidad propia.

Encapsulan valores e invariantes que forman parte del lenguaje del dominio.

Aggregates

Los Aggregates representan límites de consistencia.

Agrupan los conceptos que deben mantenerse coherentes dentro de una misma frontera de negocio.

Cada Aggregate posee exactamente un Aggregate Root.

El Aggregate Root constituye el punto de entrada para modificar el Aggregate.

Policies

Las Policies representan reglas estratégicas que gobiernan decisiones del dominio.

Conectan los Core Principles con el comportamiento operativo.

Core Principle
    ↓
Policy
    ↓
Decision

Las Policies no representan validaciones técnicas ni mecanismos de infraestructura.

Specifications

Las Specifications representan condiciones evaluables del dominio.

Responden a preguntas concretas.

Specification
    ↓
¿Se cumple esta condición?

Pueden ser utilizadas y combinadas por otras partes del dominio sin contener por sí mismas la estrategia que gobierna una decisión.

Domain Services

Los Domain Services representan comportamiento de negocio que no pertenece naturalmente a una única Entity, Value Object o Aggregate.

Son una herramienta de coordinación del dominio y deberán utilizarse únicamente cuando exista una responsabilidad real que justifique su existencia.

Domain Events

Los Domain Events representan hechos relevantes que ya han ocurrido.

Constituyen parte de la memoria del dominio y permiten mantener trazabilidad sobre acontecimientos significativos.

Domain Behavior
    ↓
Domain Event
    ↓
Domain History
Repositories

Los Repositories representan contratos de persistencia de Aggregates.

Permiten al dominio recuperar y persistir Aggregates sin conocer la tecnología utilizada.

Domain
    ↓
Repository Contract
    ↓
Infrastructure
    ↓
Persistence Technology

Los Repositories no representan acceso universal a datos.

Relationship Model

Las categorías no existen de forma aislada.

Su relación conceptual principal es:

Entities
    │
    ├── use ──→ Value Objects
    │
    └── belong to ──→ Aggregates


Aggregates
    │
    ├── protect ──→ Invariants
    │
    ├── apply ──→ Policies
    │
    ├── evaluate ──→ Specifications
    │
    └── produce ──→ Domain Events


Domain Services
    │
    ├── coordinate ──→ Aggregates
    ├── use ──→ Policies
    └── evaluate ──→ Specifications


Repositories
    │
    └── persist ──→ Aggregates

Esta representación es conceptual.

No implica que todas las relaciones deban existir en todos los casos.

Cada relación deberá justificarse por el dominio.

Strategic Decision Flow

Una de las responsabilidades fundamentales del modelo es representar cómo los principios estratégicos de CollectionHub se convierten en decisiones.

El flujo conceptual será:

Core Principles
        ↓
Strategic Intent
        ↓
Policies
        ↓
Specifications
        ↓
Domain Decision
        ↓
Aggregate / Entity / Domain Service
        ↓
Domain Event
        ↓
Domain History

Este flujo deberá permanecer trazable.

Una decisión importante del dominio deberá poder explicar qué reglas y principios contribuyeron a producirla.

Strategic Priority Principle

CollectionHub establece como principio estratégico la priorización temprana de las piezas de mayor dificultad.

Este principio tiene dos ámbitos de aplicación:

Collection Strategy
        ↓
Priorizar las piezas más difíciles de conseguir
        ↓
Reducir el riesgo estratégico de la colección

y:

Piece Acquisition
        ↓
Priorizar las oportunidades más difíciles
        ↓
Reducir la presión futura sobre la estrategia de compra

Este principio no deberá implementarse directamente como lógica dispersa.

Su evolución futura deberá seguir el flujo:

Core Principle
        ↓
Policy
        ↓
Specification
        ↓
Decision
        ↓
Domain Behavior

La implementación concreta deberá esperar hasta que el modelo correspondiente haya sido definido.

Collection-Level Strategy

El modelo deberá permitir representar decisiones estratégicas a nivel de colección.

Una colección podrá evaluarse teniendo en cuenta, entre otros factores:

Dificultad de adquisición.
Prioridad estratégica.
Estado de completitud.
Disponibilidad.
Objetivos.
Capital disponible.
Oportunidades existentes.

Estos conceptos deberán modelarse individualmente antes de establecer relaciones definitivas entre ellos.

Piece-Level Strategy

El principio de priorización también deberá poder aplicarse a nivel de pieza individual.

Esto significa que CollectionHub no deberá limitarse a preguntar:

¿Qué colección debería priorizar?

También deberá poder responder:

¿Qué pieza concreta debería intentar conseguir primero?

La estrategia de adquisición de una pieza deberá poder considerar su dificultad, oportunidad, precio objetivo y contexto estratégico.

Los criterios definitivos deberán definirse durante el modelado concreto.

Explainability

Las decisiones relevantes del dominio deberán ser explicables.

El sistema deberá poder establecer conceptualmente:

Decision
    ↓
Why?
    ↓
Policy
    ↓
Which condition?
    ↓
Specification
    ↓
Which principle?
    ↓
Core Principle

La explicabilidad constituye una propiedad fundamental del modelo.

Una decisión estratégica que no pueda justificarse deberá considerarse incompleta.

Traceability

Los conceptos importantes deberán mantener trazabilidad entre niveles.

La trazabilidad esperada será:

Principle
    ↓
Policy
    ↓
Specification
    ↓
Domain Behavior
    ↓
Domain Event

Esto permitirá reconstruir por qué una decisión fue tomada y qué acontecimiento produjo.

Modeling Boundaries

Cada concepto deberá pertenecer a la categoría que mejor represente su naturaleza.

No deberá utilizarse una categoría para solucionar problemas que pertenecen a otra.

Ejemplo:

Identity
    → Entity


Meaningful Value
    → Value Object


Consistency Boundary
    → Aggregate


Strategic Rule
    → Policy


Evaluable Condition
    → Specification


Cross-Concept Behavior
    → Domain Service


Occurred Business Fact
    → Domain Event


Aggregate Persistence
    → Repository
Separation of Concerns

El modelo deberá evitar la concentración artificial de responsabilidades.

La distribución esperada será:

Entity
    → Identity + Behavior


Value Object
    → Meaning + Value + Rules


Aggregate
    → Consistency + Invariants


Policy
    → Strategy


Specification
    → Condition


Domain Service
    → Cross-Concept Behavior


Domain Event
    → Historical Fact


Repository
    → Aggregate Persistence
Modeling Before Implementation

Ningún concepto importante del dominio deberá implementarse antes de haber sido suficientemente modelado.

El orden esperado será:

Concept Identified
    ↓
Concept Documented
    ↓
Relationships Defined
    ↓
Invariants Defined
    ↓
Behavior Defined
    ↓
Reviewed
    ↓
Approved
    ↓
Implemented

El código no deberá utilizarse como mecanismo para descubrir accidentalmente el modelo.

Evolution

El modelo es evolutivo.

La aparición de nueva información podrá provocar:

Nuevos conceptos.
Nuevas relaciones.
Refinamiento de conceptos existentes.
Eliminación de conceptos.
División de conceptos.
Consolidación de conceptos.

Toda evolución deberá mantener la coherencia con Foundation, Core Principles y Conceptual Domain.

Model Integrity

La integridad del modelo tendrá prioridad sobre la velocidad de implementación.

Cuando una nueva funcionalidad contradiga el modelo existente, deberá revisarse primero el modelo.

No deberá introducirse una excepción técnica simplemente para acelerar una implementación.

Quality Gate

Antes de considerar completo un modelo conceptual deberá verificarse:

 Los conceptos están claramente identificados.
 Cada concepto pertenece a una categoría justificada.
 Las relaciones están documentadas.
 Las invariantes están identificadas.
 Las Policies están trazadas hasta los principios.
 Las Specifications representan condiciones reales.
 Los Aggregates protegen límites de consistencia.
 Los Domain Events representan hechos relevantes.
 Los Repositories respetan los Aggregates.
 El modelo es independiente de la tecnología.
 Las decisiones relevantes son explicables.
Anti-Patterns

El modelo no deberá utilizarse para:

Reproducir la estructura de la base de datos.
Diseñar tablas antes que conceptos.
Diseñar clases antes que comportamiento.
Crear abstracciones técnicas sin significado de negocio.
Convertir cada requisito en una Entity.
Convertir cada condición en una Policy.
Convertir cada consulta en una Specification.
Convertir cada operación en un Domain Service.
Convertir cada cambio de datos en un Domain Event.
Golden Rule

El modelo debe representar el dominio tal como necesita ser comprendido por el negocio, no tal como resulta más cómodo implementarlo.

Final Objective

El objetivo de 01_MODELING es construir una representación del dominio suficientemente precisa como para que la implementación futura pueda derivarse de ella sin introducir decisiones fundamentales que todavía no hayan sido tomadas.

Cuando el modelado esté completo:

Business Knowledge
        ↓
Domain Model
        ↓
Implementation

deberá constituir una cadena continua y trazable.

El código será entonces una consecuencia del modelo, no su sustituto.