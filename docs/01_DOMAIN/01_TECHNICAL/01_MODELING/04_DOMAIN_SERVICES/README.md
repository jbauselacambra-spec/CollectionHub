# Domain Services

Version: 1.0

Status: APPROVED

---

# Purpose

La carpeta `04_DOMAIN_SERVICES` contiene la definición de los Domain Services que forman parte del dominio de CollectionHub.

Un Domain Service representa comportamiento del negocio que no pertenece naturalmente a una única Entity, Value Object o Aggregate y que requiere coordinar varios conceptos del dominio.

Un Domain Service no representa un servicio técnico ni un caso de uso de aplicación.

---

# Scope

Esta carpeta contiene exclusivamente documentos correspondientes a Domain Services identificados y aprobados dentro del modelo de dominio.

Cada Domain Service deberá disponer de su propio documento.

---

# Relationship with Standards

Todo Domain Service documentado en esta carpeta deberá cumplir:

`00_STANDARDS/04_DOMAIN_SERVICE_STANDARD.md`

El estándar técnico constituye la autoridad para determinar cómo debe documentarse y modelarse un Domain Service.

---

# Domain Responsibility

Un Domain Service deberá existir únicamente cuando exista comportamiento real del negocio que no tenga un propietario natural dentro del modelo.

Antes de crear un Domain Service deberá evaluarse siempre, en este orden:

```text
Value Object
    ↓
Entity
    ↓
Aggregate
    ↓
Domain Service
```

El Domain Service constituye una solución posterior, no una primera opción.

---

# Scope of Responsibility

Un Domain Service puede coordinar:

- Entities.
- Aggregates.
- Value Objects.
- Policies.
- Specifications.

Su responsabilidad debe limitarse al comportamiento de dominio que justifica su existencia.

---

# Stateless Nature

Los Domain Services deberán ser conceptualmente stateless.

No deberán mantener estado persistente propio.

El estado relevante pertenece a los conceptos del dominio que el servicio coordina.

---

# Modeling Rules

Todo Domain Service deberá:

- Representar comportamiento real del negocio.
- Tener un propósito claramente definido.
- Utilizar el lenguaje ubicuo.
- Mantenerse independiente de la infraestructura.
- Respetar los límites de los Aggregates.
- Evitar asumir responsabilidades de Entities.
- Evitar asumir responsabilidades de Aggregates.
- Respetar Foundation.
- Respetar el Domain Conceptual.
- Respetar `04_DOMAIN_SERVICE_STANDARD.md`.

---

# Collaboration

Cuando un Domain Service coordine varios Aggregates, deberá respetar estrictamente sus límites de consistencia.

No deberá modificar directamente el estado interno de un Aggregate.

Las modificaciones deberán realizarse mediante el comportamiento expuesto por sus respectivos Aggregate Roots.

---

# Policies and Specifications

Los Domain Services podrán utilizar Policies y Specifications para evaluar reglas y criterios del negocio.

No deberán duplicar el conocimiento que ya está formalizado en dichos conceptos.

La responsabilidad deberá mantenerse distribuida en el lugar correcto.

---

# Events

Un Domain Service podrá provocar hechos relevantes del dominio como consecuencia de una operación.

Cuando corresponda, dichos hechos deberán expresarse mediante Domain Events.

El Domain Service no deberá utilizar eventos como mecanismo técnico de comunicación.

---

# Document Lifecycle

Un Domain Service seguirá el siguiente ciclo:

```text id="qg1n5v"
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
```

Ningún Domain Service deberá considerarse parte oficial del modelo hasta alcanzar el estado `Approved`.

---

# Quality Gate

Antes de aprobar un Domain Service deberá verificarse:

- [ ] Representa comportamiento real del negocio.
- [ ] No pertenece naturalmente a una Entity.
- [ ] No pertenece naturalmente a un Value Object.
- [ ] No pertenece naturalmente a un Aggregate.
- [ ] Sus responsabilidades están claramente delimitadas.
- [ ] Sus colaboradores están identificados.
- [ ] Respeta los límites de consistencia.
- [ ] No contiene responsabilidades técnicas.
- [ ] Respeta `04_DOMAIN_SERVICE_STANDARD.md`.
- [ ] Respeta Foundation y Core Principles.

---

# Anti-Patterns

No deberán crearse Domain Services:

- Para contener lógica que no se sabe dónde colocar.
- Para centralizar toda la lógica del dominio.
- Para implementar CRUD.
- Para encapsular acceso a bases de datos.
- Para ejecutar casos de uso.
- Para sustituir Entities anémicas.
- Para sustituir Aggregate Roots.
- Para envolver servicios técnicos.

Un Domain Service que empieza a acumular responsabilidades deberá ser revisado.

---

# Golden Rule

> Un Domain Service solo existe cuando el comportamiento pertenece al dominio, pero no tiene un propietario natural dentro de una Entity, Value Object o Aggregate.

Si existe un lugar natural para la lógica, debe permanecer allí.

---

# Expected Outcome

Esta carpeta debe proporcionar un conjunto pequeño, cohesionado y justificable de Domain Services que permitan expresar comportamientos complejos del negocio sin concentrar artificialmente el conocimiento del dominio en servicios generales.