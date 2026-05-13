---
title: "Reglas de Pedidos"
tags:
  - dominio
  - pedidos
  - pagos
  - entregas
  - reglas-negocio
---

# Reglas de Pedidos, Pagos y Entregas

> [!info] Reglas de negocio para el ciclo de vida de pedidos online y su interaccion con stock y pagos.

---

## Reglas de pedidos

- Un pedido online debe tener al menos un producto.
- Un pedido debe estar asociado a una sucursal.
- Un pedido puede ser de retiro o envio.
- Un pedido no puede pasar a "listo para retirar" si no fue preparado.
- Un pedido no puede marcarse como entregado si no fue pagado, salvo configuracion explicita.
- Un pedido cancelado debe liberar stock reservado si corresponde.

## Reglas de pagos

- Un pedido puede tener pago pendiente, aprobado, rechazado o cancelado.
- El sistema no almacena datos sensibles de tarjetas.
- El link de pago puede estar simulado o integrado mediante adaptador.
- Una venta presencial puede registrar QR, efectivo, transferencia u otro metodo configurado.

## Reglas de entregas

- Un pedido con retiro se completa cuando el cliente lo retira en sucursal.
- Un pedido con envio se completa cuando se marca como entregado.
- La logistica externa queda fuera del alcance del MVP (se gestiona manualmente por el comercio).

---

## Flujo de compra online con retiro

```text
Cliente consulta catalogo
    ↓
Selecciona productos
    ↓
Selecciona retiro en sucursal
    ↓
Sistema valida stock (sin reservar)
    ↓
Sistema genera pedido (PENDIENTE_PAGO)
    ↓
Cliente paga mediante link
    ↓
Sistema revalida stock y precio → crea reserva
    ↓
Pedido pasa a PAGADO
    ↓
Empleado prepara pedido (EN_PREPARACION)
    ↓
Pedido pasa a LISTO_PARA_RETIRAR
    ↓
Cliente retira
    ↓
Pedido pasa a ENTREGADO
    ↓
Sistema confirma reserva y descuenta stock definitivamente
```

## Flujo de compra online con envio

```text
Cliente consulta catalogo
    ↓
Selecciona productos
    ↓
Indica direccion de entrega
    ↓
Sistema valida stock (sin reservar)
    ↓
Sistema genera pedido (PENDIENTE_PAGO)
    ↓
Cliente paga mediante link
    ↓
Sistema revalida stock y precio → crea reserva
    ↓
Pedido pasa a PAGADO
    ↓
Empleado prepara pedido (EN_PREPARACION)
    ↓
Pedido pasa a EN_REPARTO
    ↓
Pedido pasa a ENTREGADO
    ↓
Sistema confirma reserva y descuenta stock definitivamente
```

---

## Estados de pago

Para MVP:

```text
PENDIENTE
APROBADO
RECHAZADO
CANCELADO
```

---

## Estados de pedido (MVP)

```text
PENDIENTE_PAGO
PAGADO
EN_PREPARACION
LISTO_PARA_RETIRAR
EN_REPARTO
ENTREGADO
CANCELADO
```

---

## Integracion con stock

> [!important] Para el MVP, la reserva de stock se crea recien cuando el pago simulado queda aprobado.
> Antes de aprobar el pago, el sistema debe revalidar precio, promociones y stock disponible dentro de una transaccion.

- Al crear el pedido: se valida stock disponible, pero no se reserva stock todavia.
- Al aprobar el pago: se revalida stock disponible y precio efectivo. Si siguen siendo validos, se crea la reserva de stock de la sucursal seleccionada.
- Si no hay stock al aprobar el pago: el pago queda rechazado o pendiente de revision, y el pedido no avanza a `PAGADO`.
- Al entregar/retirar: la reserva se convierte en salida definitiva de stock.
- Al cancelar: se libera la reserva de stock si ya existia.

---

> [!seealso] Notas relacionadas
> - [[Modelo de Datos]] -- entidades PedidoOnline, PedidoOnlineItem, Pago, Entrega, ReservaStock
> - [[Reglas de Stock]] -- interaccion reservas y stock
> - [[Maquina de Estados]] -- diagrama de estados de pedido
> - [[02 - Modulos/Tienda Online]] -- checkout y tracking
> - [[02 - Modulos/Backoffice]] -- gestion de pedidos
> - Volver a [[_Index]]
