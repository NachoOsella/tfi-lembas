---
title: "Reglas de Pedidos"
tags:
  - dominio
  - pedidos
  - pagos
  - reglas-negocio
---

# Reglas de Pedidos, Pagos y Entregas (MVP)

> [!info] Ordenes unificadas (POS y ONLINE). Pagos con Mercado Pago (online) o caja (presencial).

---

## Reglas de ordenes

- Una orden debe tener al menos un item.
- Una orden debe estar asociada a una sucursal.
- `type`: `POS` (presencial) u `ONLINE` (e-commerce).
- `fulfillment_type`: solo `PICKUP` en MVP. Sin envio.
- ONLINE requiere `customer_user_id` (cliente registrado).
- POS requiere `created_by_user_id` (usuario interno).
- ONLINE se crea en `PENDING_PAYMENT`. POS se crea directo en `PAID`.
- POS requiere caja abierta en la sucursal.
- Cancelacion revierte stock y actualiza payment.

## Reglas de pagos

- Toda orden tiene al menos un `payment` asociado.
- ONLINE: `provider=MERCADO_PAGO`, `method=CHECKOUT_PRO`, `cash_session_id=null`.
- POS: `provider=MANUAL`, `method` segun medio, `cash_session_id` de caja abierta.
- Pagos online se procesan via webhook de MP.
- Pagos presenciales se registran al cobrar (APPROVED directo).
- No almacenar datos sensibles de tarjetas.

## Flujo de compra online

```text
Cliente registrado inicia sesion
    ↓
Navega catalogo, agrega al carrito local
    ↓
Confirma compra → POST /api/customer/orders
    ↓
Crea order (ONLINE, PENDING_PAYMENT) y payment (PENDING)
    ↓
Cliente solicita checkout → POST /api/customer/orders/{id}/checkout/mp
    ↓
Backend crea preferencia en MP, devuelve initPoint
    ↓
Cliente paga en MP Checkout Pro
    ↓
MP envia webhook → Backend verifica y procesa
    ↓
Si APPROVED: actualiza payment, descuenta stock FEFO, order a PAID
    ↓
Si REJECTED: actualiza payment, order a PAYMENT_FAILED
    ↓
Si sin stock al aprobar: order a STOCK_CONFLICT
    ↓
Empleado prepara → PREPARING → READY
    ↓
Cliente retira → DELIVERED (no descuenta stock)
```

## Flujo de venta presencial

```text
Empleado abre caja (monto inicial de efectivo)
    ↓
Empleado inicia venta POS
    ↓
Escanea productos
    ↓
Cobra con cualquier medio (efectivo, QR, transferencia, debito, credito)
    ↓
Sistema: valida stock, descuenta FEFO, crea order POS, crea payment asociado a caja
    ↓
Empleado cierra caja
    ↓
Sistema calcula efectivo esperado vs contado
    ↓
Si hay diferencia, se exige explicacion
```

## Integracion con stock

- ONLINE: stock se valida al crear orden y se revalida al aprobar pago.
- POS: stock se descuenta al cobrar (misma transaccion).
- Si al aprobar pago online no hay stock suficiente: `STOCK_CONFLICT`.
- Cancelacion: revierte stock con `CANCELLATION_RETURN`.

---

> [!seealso] Notas relacionadas
> - [[Modelo de Datos]]
> - [[Reglas de Stock]]
> - [[Maquina de Estados]]
> - Volver a [[_Index]]
