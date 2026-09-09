# Specifications

Version: 1.0

Status: APPROVED

---

# Purpose

La carpeta `07_SPECIFICATIONS` contiene la definición de las Specifications que forman parte del dominio de CollectionHub.

Una Specification representa una condición del negocio que puede ser evaluada de forma independiente, reutilizada y combinada con otras condiciones.

Su responsabilidad es responder a una pregunta concreta del dominio.

> ¿Se cumple esta condición?

Una Specification no ejecuta acciones y no toma decisiones estratégicas por sí misma.

---

# Scope

Esta carpeta contiene exclusivamente documentos correspondientes a Specifications identificadas y aprobadas dentro del modelo de dominio.

Cada Specification deberá disponer de su propio documento.

---

# Relationship with Standards

Toda Specification documentada en esta carpeta deberá cumplir:

`00_STANDARDS/07_SPECIFICATION_STANDARD.md`

El estándar técnico constituye la autoridad para determinar cómo debe documentarse y modelarse una Specification.

---

# Business Meaning

Una Specification debe representar conocimiento real del negocio.

Ejemplos conceptuales:

```text
¿Existe presupuesto disponible?

¿El precio cumple el objetivo establecido?

¿La oportunidad tiene suficiente prioridad?

¿La adquisición es compatible con la estrategia actual?

Single Responsibility

Cada Specification deberá responder a una única pregunta del negocio.

Una Specification no deberá combinar diferentes decisiones estratégicas en una única unidad cuando dichas decisiones puedan expresarse mediante condiciones independientes.

La composición deberá utilizarse para construir condiciones más complejas a partir de conceptos simples.

Relationship with Policies

Policies y Specifications tienen responsabilidades diferentes.

Policy


¿Qué regla estratégica debemos aplicar?


        ↓


Specification


¿Se cumple esta condición?

Una Policy puede utilizar una o varias Specifications.

Una Specification no deberá contener la estrategia de la Policy que la utiliza.

Relationship with Entities and Aggregates

Entities y Aggregates pueden utilizar Specifications para evaluar condiciones del dominio.

Una Specification no deberá modificar directamente el estado de una Entity o Aggregate.

La separación conceptual será:

Specification
    ↓
Evalúa


Entity / Aggregate
    ↓
Protege y modifica su estado
Composition

Las Specifications podrán combinarse cuando el lenguaje del dominio lo permita.

Las formas principales de composición serán:

AND


OR


NOT

La composición deberá mantener significado para el negocio.

No deberá utilizarse únicamente como mecanismo técnico para construir condiciones arbitrarias.

Explainability

Una Specification deberá poder explicar el resultado de su evaluación cuando la decisión tenga relevancia para el usuario o para la estrategia del sistema.

El resultado conceptual podrá incluir:

Resultado
    ↓
Cumple / No cumple


Razón
    ↓
Explicación de la evaluación

CollectionHub deberá evitar evaluaciones relevantes que no puedan ser justificadas.

Determinism

Una Specification deberá producir un resultado coherente para un mismo conjunto de inputs y contexto.

Si una Specification depende de información externa o cambiante, dicha dependencia deberá estar explícitamente documentada y mantenerse fuera de la propia lógica de evaluación cuando corresponda.

Infrastructure Independence

Las Specifications no deberán depender directamente de:

Entity Framework.
SQL.
APIs.
Bases de datos.
Servicios externos.
Frameworks.
DTOs.
Modelos de presentación.

Una Specification representa conocimiento del dominio, no una consulta técnica.

Reusability

Una Specification deberá poder reutilizarse cuando la misma condición del negocio aparezca en diferentes contextos.

No deberán crearse múltiples Specifications que expresen exactamente la misma regla con nombres diferentes.

Specification Lifecycle

Una Specification seguirá conceptualmente el siguiente ciclo:

Candidate
    ↓
Proposed
    ↓
Reviewed
    ↓
Approved
    ↓
Evolved
    ↓
Deprecated

Ninguna Specification deberá considerarse parte oficial del modelo hasta alcanzar el estado Approved.

Quality Gate

Antes de aprobar una Specification deberá verificarse:

 Responde a una pregunta concreta del negocio.
 Representa una condición y no una decisión estratégica.
 No modifica el estado del dominio.
 Es reutilizable cuando corresponde.
 Puede componerse con otras Specifications cuando sea necesario.
 Es explicable.
 Es independiente de infraestructura.
 Utiliza lenguaje ubicuo.
 Respeta 07_SPECIFICATION_STANDARD.md.
 Respeta Foundation y Core Principles.
Anti-Patterns

No deberán crearse Specifications:

Para ejecutar acciones.
Para modificar Entities.
Para modificar Aggregates.
Para contener Policies.
Para sustituir Domain Services.
Para encapsular consultas de infraestructura.
Para construir filtros técnicos sin significado de negocio.
Para ocultar decisiones estratégicas.

Una Specification debe evaluar, no actuar.

Golden Rule

Una Specification existe para responder de forma clara, reutilizable y explicable a una pregunta concreta del negocio.

Si no puede formularse la Specification como una condición significativa del dominio, deberá revisarse su clasificación.

Expected Outcome

Esta carpeta debe proporcionar un catálogo coherente de condiciones reutilizables del dominio que permita expresar las reglas de evaluación de CollectionHub de forma clara, componible y explicable.

Las Specifications deberán servir como piezas fundamentales de conocimiento utilizadas por Policies, Aggregates y Domain Services sin introducir dependencias técnicas.