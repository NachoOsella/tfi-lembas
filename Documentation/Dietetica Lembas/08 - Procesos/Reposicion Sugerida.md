---
title: "Reposicion Sugerida"
tags:
  - procesos
  - reposicion
  - stock-bajo
---

# Proceso: Reposicion Sugerida

> [!info] El sistema detecta productos con bajo stock comparando `SUM(stock_lots.quantity_available)` contra `products.minimum_stock`.

---

## Logica

```text
Para cada producto activo:
  stock_actual = SUM(quantity_available) FROM stock_lots WHERE product_id = X AND branch_id = Y

  Si stock_actual < products.minimum_stock:
    → Recomendar reposicion
    → Urgencia = (minimum_stock - stock_actual) * factor_rotacion
```

## Endpoint

```text
GET /api/admin/recommendations
  → Filtra por type = "LOW_STOCK"
  → Incluye: producto, stock actual, minimo, cantidad sugerida
```

---

> [!seealso] Volver a [[08 - Procesos/_Index]]
