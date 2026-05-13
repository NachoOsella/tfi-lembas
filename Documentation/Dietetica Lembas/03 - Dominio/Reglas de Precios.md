---
title: "Reglas de Precios"
tags:
  - dominio
  - precios
  - promociones
  - reglas-negocio
---

# Reglas de Precios y Promociones

> [!info] Reglas de negocio para precios base globales, promociones y ofertas por vencimiento.

---

## Precio base global

El precio base del producto pertenece a la empresa, no a una sucursal.

```text
Producto: Granola 500g
Precio base anterior: $4.500
Precio base nuevo: $4.900

Impacta en:
- venta presencial
- catalogo online
- etiquetas de precios
- reportes
- calculo de margenes
```

> [!important] Un cambio de precio base impacta en sucursales, venta presencial, e-commerce, etiquetas y reportes.

---

## Excepciones al precio base

El sistema permite excepciones controladas mediante **promociones**, no mediante duplicacion desordenada de precios.

```text
Precio base global: $4.900

Sucursal Centro:
- precio efectivo: $4.900

Sucursal Nueva Cordoba:
- oferta por vencimiento: $4.200
- stock promocional: 3 unidades
```

No se cambio el precio base. Se aplico una promocion puntual sobre el producto en esa sucursal.

---

## Precio efectivo

El precio que ve el cliente se calcula a partir del precio base y promociones aplicables:

```text
precio_efectivo = precio_base - descuento_aplicable
```

El descuento aplicable depende de:
- Sucursal seleccionada
- Fecha actual
- Canal de venta
- Sucursal seleccionada
- Fecha actual
- Canal de venta (presencial u online)
- Promocion manual activa por producto o sucursal
- Cantidad promocional disponible (si aplica)

---

## Reglas de precios

- El precio de venta puede calcularse a partir de costo + margen sugerido.
- El administrador debe aprobar cambios masivos de precios.
- Los cambios de precio deben guardar historial.
- Las ofertas temporales deben tener fecha de inicio y fin.
- Una oferta vencida no debe mostrarse como activa.

## Reglas de promociones

- Para el MVP, las promociones se crean manualmente y pueden ser globales o por sucursal.
- Las promociones por vencimiento nacen de alertas de lotes proximos a vencer, pero la aprobacion y configuracion son manuales.
- Una promocion por vencimiento no debe mostrarse en sucursales que no tengan stock disponible del producto promocionado.
- La promocion automatica por lote queda fuera del MVP para evitar complejidad en el checkout.
- Los cambios de precio y promociones deben registrar fecha, usuario responsable y motivo.

---

## Ejemplo completo

```text
Producto: Leche vegetal
Precio base global: $2.000

Sucursal Centro:
- Lote A: 2 unidades, vence pronto
- Lote B: 10 unidades, sin oferta
- Promocion manual aprobada: 20% off sobre el producto en la sucursal

Sucursal Nueva Cordoba:
- Lote C: 5 unidades, sin oferta
- Sin promocion activa
```

Resultado en e-commerce:

```text
Cliente eligio Sucursal Centro:
- ve la promocion activa
- precio promocional: $1.600

Cliente eligio Sucursal Nueva Cordoba:
- no ve la promocion
- precio: $2.000
```

---

> [!seealso] Notas relacionadas
> - [[Modelo de Datos]] -- entidades PrecioProducto, Promocion, PromocionAplicacion
> - [[Reglas de Stock]] -- interaccion stock-promociones
> - [[02 - Modulos/Backoffice]] -- actualizacion manual de costo por producto-proveedor
> - Volver a [[_Index]]
