# ADR-001: Nube AWS (us-east-1 / us-east-2) con DR por niveles de criticidad

- **Estado:** Propuesto
- **Fecha:** 2026-11-15

## Contexto

AWS no tiene región en Ecuador y el reto exige baja latencia, alta disponibilidad y una estrategia de DR. No todos los servicios necesitan el mismo tiempo de recuperación.

## Decisión

Región principal **us-east-1** y DR en **us-east-2**, con CloudFront en el borde y Direct Connect hacia el core on-premise. DR por niveles: **T0** (identidad, transferencias, auditoría) con RPO≈0 y RTO≤15 min; T1–T3 con estrategias más económicas.

## Alternativas descartadas

- **São Paulo**: De las regiones más caras; la ruta desde Ecuador no suele ser más corta.
- **Chile**: Aún no tiene disponibilidad general; se re-evaluará.
- **Activo-activo multi-región**: Costo alto y riesgo de inconsistencia financiera.

## Consecuencias

- ✅ Catálogo completo de servicios y menor costo.
- ✅ El costo del DR es proporcional a la criticidad.
- ⚠️ Datos fuera del país: se requiere contrato de encargo y cifrado con llaves del banco (LOPDP).
- ⚠️ La latencia real debe medirse antes de producción.
