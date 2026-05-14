---
title: "Reglas de Precios"
tags:
  - dominio
  - precios
  - promociones
  - reglas-negocio
---

# Reglas de Precios (MVP Simplificado)

> [!info] Precio de venta directo en el producto. Costo por proveedor. Promociones opcionales y simples.

---

## Precio de venta

El precio de venta del producto vive directamente en `products.sale_price`.

```text
Producto: Granola 500g
Precio de venta: $4.900
Costo actual: $3.200 (desde supplier_products)
Margen: 53%
```

### Cambio de precio

Cuando el admin cambia el precio, se registra en `audit_logs`. No hay tabla `price_history`.

---

## Costo por proveedor

El costo de reposicion se registra por producto y proveedor en `supplier_products.current_cost`. En el MVP se ingresa manualmente.

```text
Producto: Granola 500g
Proveedor A: $3.200 (preferido)
Proveedor B: $3.450
```

---

## Promociones (opcional, simple)

Si se implementan, solo soportan:

- Descuento fijo (`FIXED_AMOUNT`)
- Descuento porcentual (`PERCENTAGE`)
- Por producto
- Por fechas (starts_at, ends_at)

Sin cupones, combos, promociones por categoria ni reglas acumulables.

---

> [!seealso] Notas relacionadas
> - [[Modelo de Datos]] -- entidades products, product_promotions, supplier_products
> - [[Reglas de Stock]] -- interaccion stock-promociones
> - Volver a [[_Index]]
