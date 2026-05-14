---
title: "Descuento de Stock (FEFO)"
tags:
  - procesos
  - stock
  - transaccional
  - fefo
---

# Proceso: Descuento FEFO al Confirmar Pago

> [!info] Sin reservas. El stock se descuenta cuando el pago queda APPROVED (online via MP webhook o POS al cobrar).

---

## Flujo

```mermaid
%%{init: {"theme": "base", "themeVariables": {"darkMode": true, "background": "#1d2021", "primaryColor": "#32302f", "primaryTextColor": "#d4be98", "primaryBorderColor": "#d4be98", "lineColor": "#d4be98", "secondaryColor": "#32302f", "tertiaryColor": "#1d2021", "textColor": "#d4be98", "noteBkgColor": "#32302f", "noteTextColor": "#d4be98", "noteBorderColor": "#d8a657", "actorBkg": "#32302f", "actorTextColor": "#d4be98", "actorLineColor": "#d4be98", "signalColor": "#7daea3", "signalTextColor": "#d4be98", "labelBoxBkgColor": "#32302f", "labelBoxBorderColor": "#d4be98", "labelTextColor": "#d4be98", "loopTextColor": "#d4be98"}}%%
sequenceDiagram
    participant BE as Backend
    participant DB as PostgreSQL

    Note over BE,DB: ONLINE: WEBHOOK MP APPROVED
    BE->>BE: @Transactional
    BE->>DB: SELECT stock_lots FOR UPDATE (FEFO order)
    BE->>BE: Calcular lotes a descontar
    BE->>DB: UPDATE stock_lots SET quantity_available -= ?
    BE->>DB: INSERT stock_movement (type=ONLINE_SALE)
    BE->>DB: UPDATE payment (status=APPROVED)
    BE->>DB: UPDATE order (status=PAID)

    Note over BE,DB: POS: AL COBRAR
    BE->>BE: @Transactional
    BE->>DB: SELECT stock_lots FOR UPDATE (FEFO order)
    BE->>DB: UPDATE stock_lots SET quantity_available -= ?
    BE->>DB: INSERT stock_movement (type=POS_SALE)
    BE->>DB: INSERT payment (status=APPROVED, cash_session_id)
    BE->>DB: INSERT order (type=POS, status=PAID)

    Note over BE,DB: ENTREGAR (no toca stock)
    BE->>DB: UPDATE order SET status=DELIVERED

    Note over BE,DB: CANCELAR (revierte)
    BE->>DB: UPDATE stock_lots SET quantity_available += ?
    BE->>DB: INSERT stock_movement (type=CANCELLATION_RETURN)
    BE->>DB: UPDATE payment (status=CANCELLED)
    BE->>DB: UPDATE order (status=CANCELLED)
```

## Tabla de estado del stock

| Momento | Lote A | Lote B | Total |
|---|---|---|---|
| Sin orden | 10 | 5 | 15 |
| Pago aprobado (descuento 3 de A) | 7 | 5 | 12 |
| Entregado | 7 | 5 | 12 |
| Cancelado (reversion a A) | 10 | 5 | 15 |

## Reglas

| Regla | Descripcion |
|---|---|
| Stock se descuenta al aprobar pago | No al crear orden ni al entregar |
| Descuento sigue FEFO | Lote que vence primero se descuenta primero |
| Cancelacion revierte a los mismos lotes | stock_movements originales como referencia |
| Si al aprobar pago no hay stock | Order a STOCK_CONFLICT, revision manual |

---

> [!seealso] Notas relacionadas
> - [[03 - Dominio/Reglas de Stock]]
> - [[Cancelacion Pedido]]
> - Volver a [[08 - Procesos/_Index]]
