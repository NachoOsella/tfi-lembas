---
title: "Promocion por Vencimiento (opcional)"
tags:
  - procesos
  - promocion
  - vencimiento
  - oferta
---

# Proceso: Promocion por Vencimiento (Opcional en MVP)

> [!info] Las promociones son opcionales en el MVP. Si se implementan, se limitan a descuento fijo o porcentual por producto y fechas. Sin cupones, combos ni reglas acumulables.

---

## Flujo basico

```text
Sistema detecta lotes con vencimiento proximo (< 30 dias)
    ↓
Admin ve alerta en el dashboard
    ↓
Admin crea promocion manual para el producto (descuento % o fijo)
    ↓
Promocion activa entre fecha_inicio y fecha_fin
    ↓
Cliente ve precio con descuento en la sucursal correspondiente
```

## Descuentos sugeridos (logica simple)

```text
dias < 7  -> 50% off
dias < 15 -> 30% off
dias < 30 -> 20% off
```

---

> [!seealso] Volver a [[08 - Procesos/_Index]]
