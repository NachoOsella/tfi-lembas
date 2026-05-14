---
title: "Modulo de Recomendaciones"
tags:
  - modulo
  - recomendaciones
  - reglas
---

# Modulo de Recomendaciones (MVP Simplificado)

> [!info] Recomendador basado en reglas sobre datos operativos. Sin IA generativa ni machine learning.

---

## Tipos de recomendacion

### Para el administrador

```text
- Productos con stock < minimo → sugerir reposicion
- Lotes proximos a vencer → sugerir promocion
- Productos con alta rotacion → revisar stock
- Productos sin movimiento → revisar si siguen activos
```

### Para el cliente (opcional en MVP)

```text
- Productos similares en misma categoria
- Productos con stock disponible
```

## Implementacion

Endpoint unico: `GET /api/admin/recommendations`

Logica en `RecommendationService` con queries directas a la BD. Sin cache ni tablas auxiliares.

---

## Reglas de diseno

- No inventar productos (solo datos reales)
- No recomendar sin stock
- No exponer datos internos (costos, margenes)

---

> [!seealso] Volver a [[_Index]]
