# Repositories

Version: 1.0

Status: APPROVED

---

# Purpose

La carpeta `08_REPOSITORIES` contiene la definición de los Repositories que forman parte del dominio de CollectionHub.

Un Repository representa un contrato mediante el cual el dominio puede recuperar y persistir Aggregates sin conocer el mecanismo tecnológico utilizado para almacenarlos.

El Repository pertenece al modelo de dominio como abstracción.

La implementación concreta pertenece a Infrastructure.

---

# Scope

Esta carpeta contiene exclusivamente documentos correspondientes a Repositories identificados y aprobados dentro del modelo de dominio.

Cada Repository deberá disponer de su propio documento.

---

# Relationship with Standards

Todo Repository documentado en esta carpeta deberá cumplir:

`00_STANDARDS/08_REPOSITORY_STANDARD.md`

El estándar técnico constituye la autoridad para determinar cómo debe documentarse y modelarse un Repository.

---

# Aggregate Ownership

Cada Repository deberá estar asociado a un único Aggregate Root.

La relación conceptual será:

```text
Repository
    ↓
Aggregate Root
    ↓
Aggregate

Domain Perspective

Desde la perspectiva del dominio, un Repository expresa una necesidad:

"Necesito recuperar o persistir este Aggregate."

El dominio no necesita conocer:

Cómo se almacena.
Dónde se almacena.
Qué base de datos se utiliza.
Qué ORM se utiliza.
Cómo se ejecutan las consultas.
Cómo se serializan los datos.

Estas decisiones pertenecen a Infrastructure.

Responsibilities

Un Repository puede expresar operaciones conceptuales como:

Recuperar un Aggregate.
Recuperar un Aggregate mediante su identidad.
Persistir un Aggregate.
Determinar conceptualmente si un Aggregate existe cuando el dominio lo necesita.

Las operaciones deberán estar expresadas en términos del dominio.

Non-Responsibilities

Un Repository no deberá:

Contener lógica de negocio.
Ejecutar Policies.
Evaluar Specifications como responsabilidad propia.
Ejecutar Domain Services.
Generar DTOs.
Construir View Models.
Generar informes.
Realizar consultas analíticas.
Exponer detalles de persistencia.
Coordinar casos de uso.
Relationship with Aggregates

El Repository trabaja con Aggregates completos.

No deberá utilizarse para recuperar o persistir arbitrariamente Entities o Value Objects pertenecientes a un Aggregate.

El Aggregate Root constituye la frontera de acceso.

Relationship with Specifications

Una Specification puede expresar una condición del dominio.

Un Repository puede necesitar utilizar una Specification cuando la recuperación de un Aggregate requiera una condición de dominio explícita.

Sin embargo, la Specification no deberá convertirse en una consulta técnica.

La traducción de la condición a mecanismos de persistencia pertenece a Infrastructure.

Infrastructure Boundary

La separación deberá mantenerse de la siguiente forma:

DOMAIN


Repository Contract
        │
        │
        ▼
INFRASTRUCTURE


Repository Implementation
        │
        ▼
Persistence Technology

La dependencia deberá apuntar siempre hacia el dominio.

El dominio no deberá depender de Infrastructure.

Modeling Rules

Todo Repository deberá:

Gestionar un único Aggregate.
Tener un propósito claramente definido.
Utilizar lenguaje ubicuo.
Exponer únicamente necesidades de persistencia relevantes para el dominio.
Ser independiente de la tecnología.
Respetar los límites del Aggregate.
Mantener una interfaz conceptual mínima.
Respetar Foundation.
Respetar el Domain Conceptual.
Respetar 08_REPOSITORY_STANDARD.md.
Query Responsibility

Los Repositories no deberán convertirse en mecanismos generales de consulta.

Cuando el sistema necesite:

Reporting.
Analytics.
Dashboards.
Búsquedas complejas.
Proyecciones.
Consultas optimizadas para lectura.

deberá evaluarse una abstracción diferente y apropiada para dicha responsabilidad.

El Repository representa persistencia del Aggregate, no acceso universal a los datos.

Consistency

La recuperación y persistencia mediante Repository deberá respetar las invariantes y límites de consistencia del Aggregate.

El Repository no podrá utilizarse para modificar directamente partes internas del Aggregate saltándose su Aggregate Root.

Repository Lifecycle

Un Repository seguirá conceptualmente el siguiente ciclo:

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

Ningún Repository deberá considerarse parte oficial del modelo hasta alcanzar el estado Approved.

Quality Gate

Antes de aprobar un Repository deberá verificarse:

 Gestiona exactamente un Aggregate.
 El Aggregate Root está identificado.
 Su propósito está claramente definido.
 Sus operaciones están expresadas en términos del dominio.
 No contiene lógica de negocio.
 No contiene detalles de infraestructura.
 No se utiliza como mecanismo de consulta universal.
 Respeta los límites del Aggregate.
 Respeta 08_REPOSITORY_STANDARD.md.
 Respeta Foundation y Core Principles.
Anti-Patterns

No deberán crearse Repositories:

Para cada Entity.
Para cada Value Object.
Como wrappers de Entity Framework.
Como contenedores de SQL.
Para implementar reporting.
Para implementar analytics.
Para construir DTOs.
Para centralizar consultas arbitrarias.
Para evitar diseñar correctamente los Aggregates.
Golden Rule

Un Repository existe para proporcionar al dominio una abstracción de persistencia de un Aggregate, sin revelar cómo se realiza dicha persistencia.

Si el Repository empieza a parecer una capa de acceso a datos, deberá revisarse su diseño.

Expected Outcome

Esta carpeta debe proporcionar contratos de persistencia pequeños, explícitos y orientados al dominio.

Los Repositories deberán permitir que el dominio permanezca completamente independiente de la tecnología de almacenamiento y que Infrastructure pueda evolucionar sin alterar el modelo conceptual.