---
title: "Reglas de Stock"
tags:
  - dominio
  - stock
  - reglas-negocio
  - multisucursal
---

# Reglas de Stock

> [!info] Reglas de negocio para stock multisucursal, lotes, reservas y su interaccion con el e-commerce.

---

## Reglas generales de stock

- Un producto puede existir en el catalogo sin stock.
- El stock se maneja por sucursal.
- Una venta presencial descuenta stock de la sucursal donde se realizo.
- Un pedido online descuenta o reserva stock de la sucursal seleccionada.
- Todo ajuste manual de stock debe tener motivo registrado.
- El consumo interno de empleados descuenta stock y debe registrar razon.
- Las mermas y vencimientos descuentan stock con trazabilidad.
- Los productos con vencimiento cercano deben aparecer en alertas.
- Los productos con stock menor al minimo definido deben aparecer en alertas.

---

## Reglas de stock multisucursal y e-commerce

- **El e-commerce no tiene stock propio.** Consulta el stock disponible de una sucursal concreta.
- El stock se administra siempre por producto y sucursal.
- Un pedido online debe estar asociado a una **unica sucursal responsable**.
- Todos los items de un pedido online deben validarse contra el stock disponible de esa sucursal.
- Si el cliente cambia de sucursal durante la compra, el carrito debe revalidarse.
- Si un producto no tiene stock disponible en la sucursal seleccionada, no puede agregarse al carrito.
- La disponibilidad online se calcula como: `stock_disponible = stock_fisico - stock_reservado`
- Una venta presencial reduce el stock fisico de la sucursal donde se realizo.
- En el MVP, un pedido online reserva stock cuando el pago simulado queda aprobado.
- Antes de aprobar el pago, el sistema debe revalidar stock disponible dentro de una transaccion.
- Si no hay stock suficiente al aprobar el pago, el pedido no debe pasar a pagado.
- Si un pedido online se cancela, el stock reservado debe liberarse si la reserva ya existia.
- Si un pedido online se entrega o retira, la reserva se convierte en salida definitiva de stock.
- Las reservas deben tener trazabilidad y estar asociadas al pedido que las origino.
- No se permite vender mas unidades que el stock disponible.
- El sistema debe evitar condiciones de carrera mediante transacciones al confirmar ventas o reservas.

---

## Stock fisico, reservado y disponible

```text
stock_disponible = stock_fisico - stock_reservado
```

| Concepto | Descripcion |
|---|---|
| Stock fisico | Cantidad real registrada en la sucursal |
| Stock reservado | Cantidad apartada por pedidos online con pago aprobado y preparacion pendiente |
| Stock disponible | Cantidad que se puede vender en mostrador o por e-commerce |

Ejemplo:

```text
Producto: Granola 500g
Sucursal Centro

Stock fisico: 10
Stock reservado por pedidos online: 3
Stock disponible: 7
```

---

## Lotes y vencimientos

Para productos con fecha de vencimiento, el stock debe poder dividirse en lotes.

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

Aplicacion practica: si hay varios lotes disponibles, el sistema descuenta primero del lote con vencimiento mas proximo.

Para el MVP, las promociones por vencimiento se simplifican: el sistema muestra alertas de lotes proximos a vencer y permite crear una promocion manual por producto/sucursal. La promocion automatica por lote (descuento aplicado exclusivamente a unidades de un lote especifico) queda como evolucion futura.

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

## Decisiones de stock para e-commerce (MVP)

| Decision | Justificacion |
|---|---|
| Pedido online asociado a una unica sucursal | Evita split orders entre locales |
| Sin stock online separado | El e-commerce consulta stock real de sucursal |
| Stock disponible calculado por sucursal | `stock_fisico - stock_reservado` |
| Promociones por vencimiento manuales por sucursal | Reducen complejidad del checkout y siguen usando alertas de lotes |
| FEFO para descuento de stock | Prioriza lo que vence primero |
| Stock reservado al aprobar pago | Evita sobreventa sin bloquear stock durante checkout |

---

> [!seealso] Notas relacionadas
> - [[Modelo de Datos]] -- entidades StockSucursal, LoteStock, ReservaStock, MovimientoStock
> - [[Reglas de Precios]] -- promociones por vencimiento
> - [[Reglas de Pedidos]] -- interaccion stock-pedidos
> - [[02 - Modulos/Tienda Online]] -- como se muestra al cliente
> - Volver a [[_Index]]
