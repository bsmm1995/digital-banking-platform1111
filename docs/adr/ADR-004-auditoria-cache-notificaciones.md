# ADR-004: Auditoría inmutable, caché para clientes frecuentes y notificaciones por eventos

- **Estado:** Propuesto
- **Fecha:** 2026-11-15

## Contexto

El reto exige auditar toda acción del cliente, cachear a los clientes frecuentes y notificar cada movimiento (al menos 2 canales).

## Decisión

**Auditoría:** eventos → servicio en Go → base de solo inserción con hash encadenado → S3 Object Lock (WORM). **Caché:** Cache-Aside con ElastiCache Valkey (LFU) e invalidación por eventos del core. **Notificaciones:** push + email, con SMS de respaldo, disparadas por los cambios del core.

## Alternativas descartadas

- **Amazon QLDB**: Sin soporte desde 2025.
- **SMS como canal principal**: Riesgo de fraude por SMS y mayor costo.

## Consecuencias

- ✅ Cualquier manipulación de la auditoría es detectable.
- ✅ Se notifican también los movimientos de cajero y ventanilla.
- ⚠️ La caché puede quedar desactualizada unos segundos (el saldo nunca se cachea).
