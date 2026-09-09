# Ubiquitous Language

Version: 1.0

Status: APPROVED

---

# Introducción

CollectionHub utiliza un lenguaje único y compartido entre:

- Negocio
- Desarrollo
- Inteligencia Artificial
- Documentación
- Base de datos
- API
- Interfaz de usuario

Este documento define dicho lenguaje.

Toda nueva funcionalidad deberá utilizar exclusivamente los términos aquí definidos.

El objetivo es eliminar ambigüedades y conseguir que todas las capas del sistema hablen exactamente el mismo idioma.

---

# Principios

## Un concepto → Un nombre

Cada concepto del dominio tendrá un único nombre oficial.

Nunca existirán dos palabras distintas para representar el mismo concepto.

---

## Un nombre → Un significado

Cada palabra tendrá un único significado.

Nunca reutilizaremos un término para representar conceptos diferentes.

---

## El negocio gobierna el lenguaje

El lenguaje nace del dominio.

Nunca del framework.

Nunca de la tecnología.

Nunca de la base de datos.

---

## El código debe parecer negocio

El código debe poder leerse como si fuera documentación funcional.

Ejemplo:

Incorrecto

ItemService.Calculate()

Correcto

RecommendationEngine.CalculateStrategicInvestmentScore()

---

# Diccionario Oficial