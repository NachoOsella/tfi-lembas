---
title: "Estrategia de Testing"
tags:
  - referencia
  - testing
  - calidad
---

# Estrategia de Testing (MVP)

---

## Backend

### Unit tests (sin Spring, sin BD)

- FEFO: seleccion de lote, descuento parcial, descuento multiple
- Calculo de cierre: expectedCashAmount, diferencia
- Cambios de estado de orden
- Validacion de stock

### Integration tests (con Testcontainers PostgreSQL)

- Crear orden online + payment PENDING
- Crear preferencia MP (mock)
- Procesar webhook MP APPROVED + descontar stock
- Procesar webhook MP REJECTED (sin descuento)
- Webhook duplicado (idempotencia)
- Pago aprobado sin stock suficiente → STOCK_CONFLICT
- Abrir caja + registrar venta POS
- Venta POS sin caja abierta (rechazada)
- Cerrar caja con diferencia y sin explicacion (rechazada)
- Cerrar caja con diferencia y con explicacion (aceptada)
- Cancelar orden + revertir stock
- Dos cajas abiertas en misma sucursal (rechazada)

---

## Casos criticos

| Caso | Resultado esperado |
|---|---|
| Comprar mas unidades que stock disponible | Rechazado (INSUFFICIENT_STOCK) |
| Dos usuarios compran el ultimo producto | Solo uno confirma (transaccion FOR UPDATE) |
| Webhook MP duplicado | Solo se procesa una vez (idempotencia) |
| Pago MP aprobado sin stock | Order a STOCK_CONFLICT |
| Venta POS sin caja abierta | Rechazada (CASH_SESSION_REQUIRED) |
| Cerrar caja con diferencia sin explicacion | Rechazado (DIFFERENCE_REASON_REQUIRED) |
| Cerrar caja con diferencia con explicacion | Aceptado, diferencia auditada |
| CUSTOMER intenta abrir caja | Acceso denegado (403) |
| EMPLOYEE cierra caja con diferencia | Exige explicacion (auditada) |

---

> [!seealso] Volver a [[_Index]]
