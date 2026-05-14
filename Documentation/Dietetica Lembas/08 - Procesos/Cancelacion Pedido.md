---
title: "Cancelacion de Pedido"
tags:
  - procesos
  - cancelacion
  - reversion-stock
---

# Proceso: Cancelacion de Pedido y Reversion de Stock

> [!info] Al cancelar una orden pagada, el stock se revierte y el payment se actualiza.

---

## Diagrama de secuencia

```mermaid
%%{init: {"theme": "base", "themeVariables": {"darkMode": true, "background": "#1d2021", "primaryColor": "#32302f", "primaryTextColor": "#d4be98", "primaryBorderColor": "#d4be98", "lineColor": "#d4be98", "secondaryColor": "#32302f", "tertiaryColor": "#1d2021", "textColor": "#d4be98", "noteBkgColor": "#32302f", "noteTextColor": "#d4be98", "noteBorderColor": "#d8a657", "actorBkg": "#32302f", "actorTextColor": "#d4be98", "actorLineColor": "#d4be98", "signalColor": "#7daea3", "signalTextColor": "#d4be98", "labelBoxBkgColor": "#32302f", "labelBoxBorderColor": "#d4be98", "labelTextColor": "#d4be98", "loopTextColor": "#d4be98"}}%%
sequenceDiagram
    actor A as Admin
    participant FE as Frontend
    participant BE as Backend
    participant DB as PostgreSQL

    Note over A,DB: 1. CONFIRMAR CANCELACION
    A->>FE: Ingresa motivo, confirma
    FE->>BE: PATCH /api/admin/orders/{id}/cancel
    BE->>BE: @Transactional
    BE->>DB: Buscar stock_movements originales de la orden
    BE->>DB: UPDATE stock_lots (restaurar a los mismos lotes)
    BE->>DB: INSERT stock_movement (type=CANCELLATION_RETURN)
    BE->>DB: UPDATE payment (status=CANCELLED)
    BE->>DB: UPDATE order (status=CANCELLED)
    BE-->>FE: Orden cancelada
```

## Reglas de cancelacion

| Estado actual | Se puede cancelar? | Accion sobre stock | Accion sobre payment |
|---|---|---|---|
| PENDING_PAYMENT | Si | No hay stock descontado | CANCELLED |
| PAID | Si | Revertir stock | CANCELLED |
| PREPARING | Si | Revertir stock | CANCELLED |
| READY | Si (admin) | Revertir stock | CANCELLED |
| DELIVERED | No | -- | -- |
| CANCELLED | No | -- | -- |
| STOCK_CONFLICT | Si | No hay stock descontado | CANCELLED |
| PAYMENT_FAILED | Si | No hay stock descontado | CANCELLED |

---

> [!seealso] Notas relacionadas
> - [[03 - Dominio/Reglas de Stock]]
> - [[03 - Dominio/Maquina de Estados]]
> - Volver a [[08 - Procesos/_Index]]
