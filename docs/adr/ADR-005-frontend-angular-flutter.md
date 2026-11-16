# ADR-005: Frontend: Angular (web) + Flutter (móvil)

- **Estado:** Propuesto
- **Fecha:** 2026-11-15

## Contexto

El reto pide una SPA web, una app multiplataforma y 2 opciones de stack justificadas.

## Decisión

**Opción A, elegida:** Angular + Flutter. **Opción B, evaluada:** React + React Native. Los dos lenguajes se mitigan con contract-first (OpenAPI genera los clientes).

## Alternativas descartadas

- **React + React Native**: Mayor ecosistema; menos estructura y más decisiones por equipo.
- **Ionic/Capacitor**: Corre sobre WebView: más superficie de ataque.

## Consecuencias

- ✅ Estructura y seguridad integradas en Angular; compilación nativa en Flutter.
- ⚠️ Dos lenguajes; curva de aprendizaje de Angular.
