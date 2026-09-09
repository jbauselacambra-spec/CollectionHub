# Policies

Version: 1.0

Status: APPROVED

---

# Purpose

La carpeta `06_POLICIES` contiene la definición de las Policies que gobiernan las decisiones estratégicas del dominio de CollectionHub.

Una Policy representa una regla estratégica del negocio que determina cómo debe actuar el dominio ante determinadas circunstancias.

Las Policies constituyen el puente entre los Core Principles y las decisiones concretas del dominio.

---

# Scope

Esta carpeta contiene exclusivamente documentos correspondientes a Policies identificadas y aprobadas dentro del modelo de dominio.

Cada Policy deberá disponer de su propio documento.

---

# Relationship with Standards

Toda Policy documentada en esta carpeta deberá cumplir:

`00_STANDARDS/06_POLICY_STANDARD.md`

El estándar técnico constituye la autoridad para determinar cómo debe documentarse y modelarse una Policy.

---

# Strategic Foundation

Toda Policy deberá poder trazarse hasta los principios que justifican su existencia.

La relación conceptual deberá seguir:

```text
Foundation
    ↓
Core Principles
    ↓
Policy
    ↓
Decision

Decision Responsibility

Una Policy determina criterios y reglas para tomar una decisión.

No ejecuta necesariamente la acción resultante.

La separación conceptual será:

Policy
    ↓
Determina qué debería ocurrir


Specification
    ↓
Evalúa si se cumplen las condiciones


Aggregate / Entity / Domain Service
    ↓
Ejecuta el comportamiento correspondiente
Modeling Rules

Toda Policy deberá:

Representar una regla estratégica.
Tener un objetivo claramente definido.
Estar respaldada por uno o más Core Principles.
Utilizar el lenguaje ubicuo.
Ser explicable.
Ser verificable.
Ser independiente de la infraestructura.
Ser reutilizable cuando corresponda.
Mantener un ámbito claramente definido.
Respetar Foundation.
Respetar el Domain Conceptual.
Respetar 06_POLICY_STANDARD.md.
Policy Independence

Una Policy no deberá depender de:

Bases de datos.
APIs.
Frameworks.
Interfaces de usuario.
Servicios de infraestructura.
DTOs.
Mecanismos de persistencia.

La Policy expresa conocimiento estratégico, no tecnología.

Relationship with Specifications

Las Policies y Specifications tienen responsabilidades diferentes.

Policy


¿Qué regla estratégica debemos aplicar?


↓


Specification


¿Se cumple esta condición?

Una Policy podrá utilizar una o varias Specifications para evaluar sus criterios.

Una Specification no deberá contener la estrategia de la Policy.

Relationship with Aggregates

Las Policies pueden influir en decisiones que posteriormente sean ejecutadas por Aggregates.

Una Policy no deberá modificar directamente el estado interno de un Aggregate.

Relationship with Domain Services

Un Domain Service puede utilizar Policies cuando necesite aplicar criterios estratégicos durante un proceso de dominio.

El Domain Service no deberá duplicar las reglas que ya estén formalizadas en una Policy.

Explainability

Toda Policy deberá permitir explicar:

Qué decisión gobierna.
Por qué existe.
Qué principios protege.
Qué criterios utiliza.
Qué resultado puede producir.

Las decisiones estratégicas de CollectionHub no deberán convertirse en cajas negras.

Priority and Conflict Resolution

Cuando varias Policies puedan aplicarse simultáneamente, sus posibles conflictos deberán estar explícitamente documentados.

No se deberá resolver un conflicto de Policies mediante una decisión implícita en el código.

La prioridad deberá formar parte del conocimiento del dominio cuando sea relevante.

Policy Lifecycle

Una Policy seguirá conceptualmente el siguiente ciclo:

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

Ninguna Policy deberá considerarse parte oficial del modelo hasta alcanzar el estado Approved.

Quality Gate

Antes de aprobar una Policy deberá verificarse:

 Representa una regla estratégica real.
 Su objetivo está claramente definido.
 Existe una relación explícita con Core Principles.
 Su ámbito está definido.
 Sus criterios son comprensibles.
 Es explicable.
 Es verificable.
 No contiene lógica técnica.
 No duplica Specifications.
 Respeta 06_POLICY_STANDARD.md.
Anti-Patterns

No deberán crearse Policies:

Para validaciones técnicas.
Para sustituir Specifications.
Para contener lógica de infraestructura.
Para implementar casos de uso.
Para centralizar arbitrariamente lógica del dominio.
Porque exista una clase que necesite un nombre.
Para ocultar decisiones no documentadas.

Una Policy debe representar una decisión estratégica real.

Golden Rule

Una Policy existe para proteger una decisión estratégica del negocio.

Si no puede explicarse qué principio protege y qué decisión gobierna, probablemente no pertenece a esta categoría.

Expected Outcome

Esta carpeta debe proporcionar un conjunto pequeño, coherente y trazable de reglas estratégicas que permitan garantizar que las decisiones de CollectionHub sean consistentes con sus principios fundamentales.

Las Policies deberán constituir el mecanismo mediante el cual la estrategia del proyecto se convierte en comportamiento repetible y explicable.