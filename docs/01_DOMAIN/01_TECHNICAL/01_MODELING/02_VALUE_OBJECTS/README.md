# Value Objects

Version: 1.0

Status: APPROVED

---

# Purpose

La carpeta `02_VALUE_OBJECTS` contiene la definición de los Value Objects que forman parte del dominio de CollectionHub.

Un Value Object representa un concepto del dominio definido por su valor y significado, no por una identidad propia.

Los Value Objects encapsulan conocimiento y reglas relacionadas con dicho valor.

No representan tablas de base de datos.

No representan entidades.

No representan DTOs.

---

# Scope

Esta carpeta contiene exclusivamente documentos correspondientes a Value Objects identificados y aprobados dentro del modelo de dominio.

Cada Value Object deberá disponer de su propio documento.

---

# Relationship with Standards

Todo Value Object documentado en esta carpeta deberá cumplir:

`00_STANDARDS/02_VALUE_OBJECT_STANDARD.md`

El estándar técnico constituye la autoridad para determinar cómo debe documentarse y modelarse un Value Object.

---

# Relationship with Other Concepts

Un Value Object puede formar parte de:

- Entities.
- Aggregate Roots.
- Aggregates.
- Domain Services.
- Policies.
- Specifications.

Su utilización deberá estar justificada por el significado que aporta al dominio.

---

# Modeling Rules

Todo Value Object deberá:

- Representar un concepto real del dominio.
- Ser definido por su valor.
- Ser inmutable.
- Encapsular las reglas asociadas a su significado.
- Utilizar el lenguaje ubicuo.
- Ser independiente de la infraestructura.
- Evitar representar identidad.
- Mantener coherencia con el Domain Conceptual.
- Respetar los Core Principles.

---

# Identity

Un Value Object no posee identidad propia.

Dos instancias con exactamente el mismo significado deberán considerarse equivalentes aunque sean objetos diferentes.

La identidad deberá pertenecer al concepto que realmente la necesite, no al Value Object.

---

# Immutability

Los Value Objects deberán tratarse como conceptos inmutables.

Cuando el valor deba cambiar, deberá crearse un nuevo valor en lugar de modificar el existente.

Esto garantiza que las reglas asociadas al concepto permanezcan controladas.

---

# Behavior

Un Value Object no debe limitarse a almacenar datos.

Debe encapsular el comportamiento que pertenezca naturalmente al valor que representa.

La lógica asociada a un concepto deberá permanecer junto al concepto siempre que sea posible.

---

# Boundaries

Un concepto no deberá convertirse en Value Object únicamente porque parezca pequeño o porque contenga pocos datos.

Debe existir una razón de dominio para su existencia.

No deberán utilizarse Value Objects para representar:

- Entities.
- Aggregates.
- Procesos.
- Servicios.
- DTOs.
- Modelos de persistencia.

---

# Document Lifecycle

Un Value Object seguirá el siguiente ciclo:

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

Ningún Value Object deberá considerarse parte oficial del modelo hasta alcanzar el estado `Approved`.

---

# Quality Gate

Antes de aprobar un Value Object deberá verificarse:

- [ ] Representa un concepto real del dominio.
- [ ] Su identidad no es necesaria.
- [ ] Su significado está claramente definido.
- [ ] Sus invariantes están documentadas.
- [ ] Sus reglas pertenecen al propio valor.
- [ ] Su utilización está justificada.
- [ ] Respeta `02_VALUE_OBJECT_STANDARD.md`.
- [ ] Respeta Foundation y Core Principles.

---

# Anti-Patterns

No se deberán crear Value Objects:

- Para sustituir una Entity.
- Para crear simples wrappers sin significado.
- Porque una propiedad tenga un tipo primitivo.
- Para ocultar estructuras técnicas.
- Como contenedores arbitrarios de datos.
- Únicamente para mejorar la estructura de una base de datos.

---

# Golden Rule

> Un Value Object existe porque el valor que representa tiene un significado y unas reglas propias dentro del dominio.

Si el concepto no aporta significado adicional al dominio, deberá cuestionarse la necesidad de convertirlo en Value Object.

---

# Expected Outcome

Esta carpeta debe proporcionar una representación clara y coherente de los valores significativos del dominio, encapsulando su significado, invariantes y comportamiento y evitando que el conocimiento del negocio quede disperso en tipos primitivos.