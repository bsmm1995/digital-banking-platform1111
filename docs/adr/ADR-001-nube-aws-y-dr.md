# ADR-001: Nube AWS (us-east-1 / us-east-2) con DR por niveles de criticidad

- **Estado:** Propuesto
- **Fecha:** 2026-11-15

## Contexto

AWS no tiene región en Ecuador y el reto exige baja latencia, alta disponibilidad y una estrategia de DR. No todos los servicios necesitan el mismo tiempo de recuperación.

## Decisión

Región principal **us-east-1** y DR en **us-east-2**, con CloudFront en el borde y Direct Connect hacia el core on-premise. DR por niveles: **T0** (identidad, transferencias, auditoría) con RPO≈0 y RTO≤15 min; T1–T3 con estrategias más económicas. **Respaldos búnker** inmutables en **Oracle Cloud (Bogotá)**, con llaves de cifrado custodiadas fuera de AWS, como protección ante una falla total del proveedor.

## Alternativas descartadas

- **São Paulo**: De las regiones más caras; la ruta desde Ecuador no suele ser más corta.
- **Chile**: Aún no tiene disponibilidad general; se re-evaluará por catálogo y precio.
- **DR en São Paulo**: Más costo y más retraso de replicación; no reduce el riesgo de proveedor porque sigue siendo AWS.
- **Multicloud activo**: Complejidad y costo desproporcionados para el tamaño del banco; la otra nube se usa solo para respaldos.
- **Activo-activo multi-región**: Costo alto y riesgo de inconsistencia financiera.

## Consecuencias

- ✅ Catálogo completo de servicios y menor costo.
- ✅ El costo del DR es proporcional a la criticidad.
- ⚠️ Datos fuera del país: se requiere contrato de encargo y cifrado con llaves del banco (LOPDP).
- ✅ Los datos sobreviven incluso a una caída total de AWS (copia fuera del proveedor).
- ⚠️ La latencia real debe medirse antes de producción.
- ⚠️ La copia búnker protege datos, no el servicio: recuperar en otra nube tomaría días.
