# Foundation Overview

Version: 1.0

Status: APPROVED

---

# Objetivo

La carpeta **00_FOUNDATION** contiene los principios inmutables sobre los que se construye CollectionHub.

Estos documentos representan la identidad del proyecto y constituyen la base de todas las decisiones futuras.

La Foundation responde a una única pregunta:

> **¿Por qué existe CollectionHub y cuáles son los principios que nunca cambiarán?**

La Foundation **no define reglas de negocio**, **no describe la arquitectura técnica** y **no implementa funcionalidades**.

Su propósito es proporcionar una base sólida y estable sobre la que evolucionará el resto del sistema.

---

# Principios de la Foundation

Toda la Foundation debe cumplir las siguientes reglas:

- Es independiente de cualquier tecnología.
- Es independiente del lenguaje de programación.
- Es independiente del framework utilizado.
- Es independiente de la interfaz de usuario.
- Es independiente de la base de datos.
- Es independiente de la Inteligencia Artificial.

La Foundation debe permanecer estable aunque el resto del sistema evolucione.

---

# Relación entre los documentos

Los documentos de la Foundation siguen un orden lógico.

Cada documento responde a una pregunta distinta.

```
¿Por qué existe CollectionHub?
        │
        ▼
01_PURPOSE
        │
        ▼
¿Qué creemos como proyecto?
        │
        ▼
02_PROJECT_MANIFESTO
        │
        ▼
¿Qué leyes nunca deben romperse?
        │
        ▼
03_COLLECTION_CONSTITUTION
        │
        ▼
¿Qué principios guían todas las decisiones?
        │
        ▼
04_CORE_PRINCIPLES
        │
        ▼
¿Cómo diseñamos CollectionHub?
        │
        ▼
05_DESIGN_PHILOSOPHY
```

Cada documento amplía al anterior sin sustituirlo.

---

# Responsabilidades

## 01_PURPOSE

Define el propósito fundamental del proyecto.

Responde únicamente a:

> ¿Por qué existe CollectionHub?

---

## 02_PROJECT_MANIFESTO

Describe la visión y la filosofía del proyecto.

Explica qué pretende conseguir CollectionHub y cuál es su forma de entender el coleccionismo estratégico.

---

## 03_COLLECTION_CONSTITUTION

Recoge las leyes fundamentales que gobiernan el sistema.

La Constitución no explica.

La Constitución declara.

Todos los motores del sistema deberán respetarla.

---

## 04_CORE_PRINCIPLES

Define los principios universales que deben aplicarse en cualquier decisión.

Estos principios sirven como guía para el diseño, la implementación y la evolución del proyecto.

---

## 05_DESIGN_PHILOSOPHY

Describe cómo debe diseñarse CollectionHub.

Establece los criterios arquitectónicos que garantizan la coherencia del sistema a largo plazo.

---

# Qué NO pertenece a Foundation

Los siguientes conceptos pertenecen a otras áreas del proyecto.

## Domain

- Entidades
- Value Objects
- Servicios
- Eventos
- Reglas de negocio
- Modelos de decisión

---

## Architecture

- Clean Architecture
- Frontend
- Backend
- APIs
- Persistencia
- Infraestructura

---

## Product

- Funcionalidades
- Roadmap
- Backlog
- UX
- Releases

---

## AI

- Prompts
- Agentes
- Contexto
- Workflows
- Memoria

---

## Development

- Convenciones de código
- Git
- Testing
- CI/CD

---

# Dependencias

La Foundation no depende de ningún otro documento.

Sin embargo, el resto del proyecto depende de la Foundation.

```
                    Foundation
                         │
      ┌──────────────────┼──────────────────┐
      ▼                  ▼                  ▼
   Domain          Architecture         Product
      │                  │                  │
      └──────────────┬───┘                  │
                     ▼                      ▼
                    AI               Development
```

Toda nueva funcionalidad deberá ser coherente con la Foundation.

---

# Regla de Oro

Si una decisión contradice cualquiera de los documentos de la Foundation, la decisión debe considerarse incorrecta.

La Foundation constituye la fuente de verdad del proyecto.

---

# Objetivo Final

La Foundation garantiza que CollectionHub pueda evolucionar durante años manteniendo siempre una identidad coherente, una arquitectura consistente y una filosofía estable.