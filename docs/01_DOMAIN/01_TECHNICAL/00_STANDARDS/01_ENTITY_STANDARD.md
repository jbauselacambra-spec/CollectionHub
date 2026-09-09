# Entity Standard

Version: 1.0

Status: APPROVED

---

# Objetivo

Este documento define el estándar oficial para modelar entidades dentro del Dominio Técnico de CollectionHub.

Todas las entidades del proyecto deberán seguir esta estructura.

El objetivo es garantizar que el dominio mantenga un lenguaje uniforme, un comportamiento consistente y una documentación de alta calidad independientemente de quién implemente el código.

Las entidades representan conceptos del negocio con identidad propia y comportamiento significativo.

No representan tablas.

No representan DTOs.

No representan modelos de persistencia.

Representan conocimiento del dominio.

---

# Principios

Toda entidad debe cumplir los siguientes principios.

- Debe representar un concepto del negocio.
- Debe poseer identidad propia.
- Debe encapsular comportamiento.
- Debe proteger sus invariantes.
- Debe evitar estados inválidos.
- Debe expresar el lenguaje ubicuo.
- Debe ser independiente de la infraestructura.
- Debe ser independiente de la persistencia.
- Debe ser independiente del framework utilizado.

---

# Estructura Obligatoria

Toda entidad deberá documentarse utilizando exactamente las siguientes secciones.

```
# Entity Name

## Purpose

## Responsibilities

## Identity

## Lifecycle

## States

## Invariants

## Relationships

## Behaviors

## Business Rules

## Generated Events

## Collaborators

## Dependencies

## Examples

## Future Evolution
```

No deben eliminarse secciones.

Si una sección no aplica deberá justificarse.

---

# 1. Purpose

Describe por qué existe la entidad.

Debe responder únicamente a una pregunta.

> ¿Qué representa dentro del dominio?

Nunca debe explicar cómo funciona.

---

# 2. Responsibilities

Lista todas las responsabilidades de la entidad.

Cada responsabilidad debe ser coherente con el principio de responsabilidad única.

Si aparecen responsabilidades no relacionadas probablemente debamos dividir la entidad.

---

# 3. Identity

Define qué convierte a una instancia en única.

Ejemplos.

PortfolioId

CollectionId

PurchaseId

CollectibleId

Nunca utilizar propiedades de infraestructura.

---

# 4. Lifecycle

Describe el ciclo de vida completo.

Ejemplo.

Created

Active

Completed

Archived

Deleted

Debe reflejar únicamente estados del dominio.

---

# 5. States

Define los posibles estados internos.

Cada estado debe tener significado para el negocio.

Nunca para la implementación.

---

# 6. Invariants

Las invariantes representan reglas que nunca pueden romperse.

Ejemplos.

Una compra no puede tener importe negativo.

Un Portfolio siempre debe existir.

Una colección no puede contener elementos duplicados.

Las invariantes son responsabilidad exclusiva de la entidad.

---

# 7. Relationships

Describe las relaciones con otras entidades.

Debe indicar.

Tipo de relación.

Responsabilidad.

Dependencia conceptual.

Nunca relaciones de base de datos.

---

# 8. Behaviors

Lista todos los comportamientos públicos del dominio.

Ejemplo.

RegisterPurchase()

CompleteCollection()

ReserveBudget()

GenerateRecommendation()

El comportamiento debe expresarse utilizando el lenguaje ubicuo.

---

# 9. Business Rules

Describe las reglas específicas del negocio.

No incluir reglas compartidas.

Las reglas compartidas pertenecen a Policies o Specifications.

---

# 10. Generated Events

Enumera todos los eventos del dominio que puede producir la entidad.

Ejemplo.

PurchaseRegistered

CollectionCompleted

RecommendationAccepted

SaleRegistered

---

# 11. Collaborators

Indica con qué otras entidades o servicios colabora.

No describir dependencias técnicas.

Solo relaciones del dominio.

---

# 12. Dependencies

Documenta únicamente dependencias conceptuales.

Nunca:

Entity Framework

SQL Server

ASP.NET

JSON

REST

---

# 13. Examples

Incluye ejemplos reales del dominio.

No ejemplos de código.

Deben ayudar a comprender el comportamiento de la entidad.

---

# 14. Future Evolution

Enumera posibles extensiones futuras.

No representan funcionalidades comprometidas.

Solo direcciones de evolución compatibles con el dominio.

---

# Calidad Esperada

Antes de considerar terminada una entidad deberá responder afirmativamente a todas las preguntas.

□ ¿Representa un concepto del negocio?

□ ¿Tiene identidad propia?

□ ¿Posee comportamiento?

□ ¿Protege sus invariantes?

□ ¿Puede entenderse sin leer código?

□ ¿Utiliza el lenguaje ubicuo?

□ ¿Respeta la Constitución?

□ ¿Respeta los Core Principles?

□ ¿Puede implementarse en cualquier tecnología?

□ ¿Tiene una única responsabilidad?

---

# Anti-Patrones

Una entidad nunca debe convertirse en:

- DTO
- Modelo de base de datos
- Contenedor de propiedades
- Clase utilitaria
- Servicio disfrazado
- Objeto anémico

Si una entidad solo contiene datos probablemente no sea una verdadera entidad del dominio.

---

# Regla de Oro

Las entidades existen para proteger el conocimiento del negocio.

El código es únicamente una implementación de dicho conocimiento.

---

# Objetivo Final

Conseguir que todas las entidades de CollectionHub compartan una misma filosofía de diseño, un mismo lenguaje y un mismo nivel de calidad, permitiendo que el modelo de dominio evolucione de forma coherente durante toda la vida del proyecto.