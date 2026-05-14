---
title: "Reglas de Stock"
tags:
  - dominio
  - stock
  - reglas-negocio
  - multisucursal
---

# Reglas de Stock

> [!info] Reglas de negocio para stock multisucursal, lotes, vencimientos y su interaccion con ordenes.
> Modelo simplificado: `stock_lots` es la unica fuente de verdad del stock. No hay `branch_product_stock` ni `stock_reservations`. El stock disponible se calcula como `SUM(quantity_available)` de los lotes activos.

---

## Reglas generales de stock

- Un producto puede existir en el catalogo sin stock.
- El stock se maneja por sucursal.
- El stock disponible se calcula como la suma de `quantity_available` de todos los lotes activos de un producto en una sucursal.
- Una venta presencial descuenta stock de la sucursal donde se realizo, al momento del cobro.
- Un pedido online descuenta stock cuando el pago se aprueba (no antes).
- Todo ajuste manual de stock debe tener motivo registrado en un `stock_movement`.
- El consumo interno de empleados descuenta stock y debe registrar razon.
- Las mermas y vencimientos descuentan stock con trazabilidad mediante `stock_movements`.
- Los productos con vencimiento cercano deben aparecer en alertas.
- Los productos con stock menor al minimo definido deben aparecer en alertas.

---

## Reglas de stock multisucursal y e-commerce

- **El e-commerce no tiene stock propio.** Consulta el stock disponible de una sucursal concreta como suma de sus lotes.
- El stock se administra siempre por producto y sucursal.
- Una orden online debe estar asociada a una **unica sucursal responsable**.
- Todos los items de una orden online deben validarse contra el stock disponible de esa sucursal.
- Si el cliente cambia de sucursal durante la compra, el carrito debe revalidarse.
- Si un producto no tiene stock disponible en la sucursal seleccionada, no puede agregarse al carrito.
- La disponibilidad online se calcula como `SUM(stock_lots.quantity_available) WHERE product_id = ? AND branch_id = ?`.
- Una venta presencial descuenta stock directamente de los lotes de la sucursal donde se realizo.
- En el MVP, el stock se descuenta cuando el pago se aprueba (para online) o cuando se cobra (presencial).
- Antes de aprobar el pago o cobrar, el sistema debe revalidar stock disponible dentro de una transaccion con `SELECT ... FOR UPDATE`.
- Si no hay stock suficiente al aprobar el pago, la orden no debe pasar a pagada.
- Si una orden se cancela, se genera un movimiento inverso de stock (tipo `CANCELLATION_RETURN`) que restaura la cantidad a los lotes.
- No se permite vender mas unidades que el stock disponible.
- El sistema debe evitar condiciones de carrera mediante transacciones con bloqueo pesimista en `stock_lots`.

---

## Stock disponible

El stock disponible se calcula directamente desde `stock_lots`:

```text
stock_disponible = SUM(stock_lots.quantity_available)
WHERE product_id = ? AND branch_id = ?
```

No existe stock "reservado" como campo separado ni tabla de resumen. Cuando un pago se aprueba, el stock se descuenta directamente de los lotes. Si se cancela, se revierte contra los mismos lotes.

Ejemplo:

```text
Producto: Granola 500g
Sucursal Centro

Lote A: 5 unidades (vence 2026-07-10)
Lote B: 5 unidades (vence 2026-09-15)
Stock disponible: 10
```

---

## Lotes y vencimientos

Para productos con fecha de vencimiento, el stock debe dividirse en lotes.

```text
Producto: Leche vegetal
Sucursal Centro

Lote A:
- cantidad: 2
- vencimiento: 2026-05-20

Lote B:
- cantidad: 10
- vencimiento: 2026-07-10
```

Esto permite:
- Alertas de vencimiento
- Descuentos por vencimiento cercano
- Trazabilidad de perdidas
- Criterio FEFO (First Expired, First Out)

---

## Criterio FEFO

> [!important] Para productos con vencimiento, el sistema debe priorizar las unidades que vencen primero.

```text
FEFO = First Expired, First Out
```

Aplicacion practica: al descontar stock (por venta presencial o pago aprobado), el sistema descuenta primero del lote con vencimiento mas proximo.

Esto se implementa en la transaccion:

```text
BEGIN;
  SELECT id, quantity_available, expiration_date
  FROM stock_lots
  WHERE product_id = ? AND branch_id = ? AND quantity_available > 0
  ORDER BY expiration_date ASC NULLS LAST
  FOR UPDATE;

  -- Descontar del primer lote (FEFO), luego del siguiente si hace falta
  -- INSERT stock_movement
  -- INSERT/UPDATE order
COMMIT;
```

Para el MVP, las promociones por vencimiento se simplifican: el sistema muestra alertas de lotes proximos a vencer y permite crear una promocion manual por producto/sucursal.

---

## Ofertas por vencimiento

Las ofertas por vencimiento no deberian ser globales, porque dependen del stock fisico de cada sucursal.

```text
Producto: Leche vegetal
Precio base: $2.000

Sucursal Centro:
- Lote A proximo a vencer: 2 unidades vencen pronto
- Lote B normal: 10 unidades
- Promocion manual activa: 20% off sobre el producto en esta sucursal

Sucursal Nueva Cordoba:
- Sin lote proximo a vencer
- Sin promocion activa
```

En el e-commerce, si el cliente eligio Sucursal Centro, puede ver la oferta. Si eligio Nueva Cordoba, no.

---

## Stock y cancelacion de ordenes

Cuando una orden se cancela despues de haber descontado stock, se revierte el descuento:

```text
BEGIN;
  -- Buscar movimientos originales de la orden
  SELECT stock_lot_id, ABS(quantity_delta)
  FROM stock_movements
  WHERE order_id = ?;

  -- Restaurar la cantidad a los mismos lotes
  UPDATE stock_lots
  SET quantity_available = quantity_available + ?
  WHERE id = ?;

  -- Registrar movimiento inverso
  INSERT INTO stock_movements (
    product_id, branch_id, stock_lot_id, order_id, user_id,
    movement_type, quantity_delta, reason
  ) VALUES (?, ?, ?, ?, ?, 'CANCELLATION_RETURN', +?, ?);

COMMIT;
```

Este enfoque reemplaza la necesidad de una tabla separada `stock_reservations`. El movimiento de venta y su eventual cancelacion quedan registrados en `stock_movements` con trazabilidad completa.

---

## Flujo de stock en orden online

```text
Cliente crea pedido online (PENDING_PAYMENT)
    ↓
No se toca stock
    ↓
Pago confirmado por admin
    ↓
Transaccion:
  1. SELECT ... FOR UPDATE sobre stock_lots (lotes del producto en la sucursal, ordenados FEFO)
  2. Validar que SUM(quantity_available) >= cantidad solicitada
  3. UPDATE stock_lots (descontar FEFO)
  4. INSERT stock_movement (tipo ONLINE_ORDER)
  5. UPDATE order (status = PAID, payment_status = APPROVED)
    ↓
Cliente retira en sucursal
    ↓
UPDATE order (status = DELIVERED) -- no descuenta stock
    ↓
(Si se cancela antes de entregar: revertir con CANCELLATION_RETURN)
```

---

## Tipos de movimiento de stock

| Tipo | Descripcion | quantity_delta |
|---|---|---|
| `PURCHASE_ENTRY` | Ingreso por compra/proveedor | Positivo |
| `POS_SALE` | Venta presencial | Negativo |
| `ONLINE_ORDER` | Pedido online pagado | Negativo |
| `CANCELLATION_RETURN` | Cancelacion que devuelve stock | Positivo |
| `MANUAL_ADJUSTMENT` | Ajuste manual | Positivo o negativo |
| `WASTE` | Merma | Negativo |
| `INTERNAL_CONSUMPTION` | Consumo interno | Negativo |
| `TRANSFER_IN` | Transferencia desde otra sucursal (opcional) | Positivo |
| `TRANSFER_OUT` | Transferencia a otra sucursal (opcional) | Negativo |

---

## Decisiones de stock para e-commerce (MVP)

| Decision | Justificacion |
|---|---|
| Orden online asociada a una unica sucursal | Evita split orders entre locales |
| Sin stock online separado | El e-commerce consulta stock real de sucursal |
| Stock disponible calculado desde `stock_lots` | Fuente de verdad por lote, FEFO, alertas de vencimiento |
| Promociones por vencimiento manuales por sucursal | Reducen complejidad del checkout |
| FEFO para descuento de stock | Prioriza lo que vence primero |
| Stock descontado al aprobar pago | Sin tabla de reservas separada |
| Cancelacion revierte con movimiento inverso | Trazabilidad completa en `stock_movements` |
| Sin tabla de resumen (branch_product_stock) | stock_lots es la unica fuente de verdad |

---

> [!seealso] Notas relacionadas
> - [[Modelo de Datos]] -- entidades stock_lots, stock_movements
> - [[Reglas de Precios]] -- promociones por vencimiento
> - [[Reglas de Pedidos]] -- interaccion stock-ordenes
> - [[02 - Modulos/Tienda Online]] -- como se muestra al cliente
> - Volver a [[_Index]]
