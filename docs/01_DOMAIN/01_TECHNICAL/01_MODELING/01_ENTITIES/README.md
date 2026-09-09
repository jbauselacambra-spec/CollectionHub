# Entities

Version: 1.0

Status: APPROVED

---

# Purpose

La carpeta `01_ENTITIES` contiene la definición de las Entities que forman parte del dominio de CollectionHub.

Una Entity representa un concepto del dominio cuya identidad es relevante y permanece a lo largo de su ciclo de vida.

Las Entities expresan comportamiento y estado propios del dominio.

No representan tablas de base de datos.

No representan modelos de persistencia.

No representan DTOs.

---

# Scope

Esta carpeta contiene exclusivamente documentos correspondientes a Entities identificadas y aprobadas dentro del modelo de dominio.

Cada Entity deberá disponer de su propio documento.

---

# Relationship with Standards

Toda Entity documentada en esta carpeta deberá cumplir:

`00_STANDARDS/01_ENTITY_STANDARD.md`

El estándar técnico constituye la autoridad para determinar cómo debe documentarse y modelarse una Entity.

---

# Relationship with Other Concepts

Una Entity puede relacionarse con:

- Value Objects.
- Aggregate Roots.
- Otras Entities.
- Domain Services.
- Domain Events.
- Policies.
- Specifications.

Estas relaciones deberán estar explícitamente documentadas cuando sean relevantes para comprender el dominio.

---

# Modeling Rules

Toda Entity deberá:

- Tener una identidad claramente definida.
- Tener un propósito de negocio explícito.
- Utilizar el lenguaje ubicuo.
- Encapsular comportamiento propio.
- Mantener sus invariantes.
- Ser independiente de la infraestructura.
- Mantener coherencia con el Domain Conceptual.
- Respetar los Core Principles.

---

# Boundaries

Una Entity no deberá utilizarse para representar:

- Un Value Object.
- Un Aggregate completo.
- Un proceso del negocio.
- Una regla estratégica aislada.
- Una consulta.
- Un modelo de persistencia.

La clasificación deberá realizarse antes de crear el documento.

---

# Document Lifecycle

Una Entity seguirá el siguiente ciclo:

```text
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

Ninguna Entity deberá considerarse parte oficial del modelo hasta alcanzar el estado `Approved`.

---

# Quality Gate

Antes de aprobar una Entity deberá verificarse:

- [ ] Su identidad está claramente definida.
- [ ] Su propósito pertenece al dominio.
- [ ] Su comportamiento está identificado.
- [ ] Sus invariantes están documentadas.
- [ ] Sus relaciones están justificadas.
- [ ] Su existencia está respaldada por el Domain Conceptual.
- [ ] Respeta `01_ENTITY_STANDARD.md`.
- [ ] Respeta Foundation y Core Principles.

---

# Anti-Patterns

No se deberán crear Entities:

- Porque existe una tabla.
- Porque existe una pantalla.
- Porque existe un DTO.
- Porque una clase necesita un nombre.
- Para almacenar datos sin comportamiento.
- Para evitar utilizar Value Objects correctamente.

---

# Golden Rule

> Una Entity existe porque el negocio necesita reconocer y mantener la identidad de ese concepto.

Si la identidad no tiene significado para el negocio, deberá evaluarse si el concepto pertenece realmente a esta categoría.

---

# Expected Outcome

Esta carpeta debe proporcionar una representación clara y cohesionada de todas las Entities del dominio, permitiendo comprender su identidad, comportamiento, responsabilidades y relaciones sin necesidad de conocer la implementación técnica.