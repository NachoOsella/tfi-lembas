---
title: "Recomendaciones por Reglas"
tags:
  - procesos
  - recomendaciones
  - reglas
---

# Proceso: Recomendaciones del Sistema

> [!info] Recomendador basado en reglas simples sobre datos operativos. Sin IA generativa ni machine learning.

---

## Tipos de recomendacion

### Para el administrador (GET /api/admin/recommendations)

```text
- Stock disponible < stock_minimum  → "Sugerir reposicion"
- Lote vence en < 30 dias          → "Sugerir promocion o priorizar venta"
- Producto con alta rotacion       → "Revisar que no falte stock"
- Producto sin ventas recientes    → "Revisar si sigue activo"
```

### Para el cliente (futuro)

En el MVP, las recomendaciones al cliente son opcionales. Si se implementan:
- Productos similares en la misma categoria
- Productos con stock disponible en la sucursal

## Implementacion

```text
RecommendationService (logica simple en el service)
  ├── getLowStockRecommendations()     → stock_lots + products.minimum_stock
  ├── getExpiringSoonRecommendations() → stock_lots.expiration_date
  ├── getHighRotationRecommendations() → order_items agrupados
  └── getNoMovementRecommendations()   → products sin order_items recientes
```

No requiere tabla separada ni cache. Calculo en tiempo real.

---

> [!seealso] Volver a [[08 - Procesos/_Index]]
