---
title: "Cancelacion de Pedido"
tags:
  - procesos
  - cancelacion
  - liberacion-stock
---

# Proceso: Cancelacion de Pedido y Liberacion de Stock

> [!info] Cuando un pedido se cancela, el stock reservado se libera automaticamente.
>
> Para el MVP: si el pedido ya fue pagado, el pago permanece como APROBADO. El sistema no ejecuta reembolsos automaticos. Se registra una accion administrativa pendiente para que el administrador gestione el reintegro manualmente.

## Diagrama de secuencia

```mermaid
%%{init: {"theme": "base", "themeVariables": {"darkMode": true, "background": "#1d2021", "primaryColor": "#32302f", "primaryTextColor": "#d4be98", "primaryBorderColor": "#d4be98", "lineColor": "#d4be98", "secondaryColor": "#32302f", "tertiaryColor": "#1d2021", "textColor": "#d4be98", "noteBkgColor": "#32302f", "noteTextColor": "#d4be98", "noteBorderColor": "#d8a657", "actorBkg": "#32302f", "actorTextColor": "#d4be98", "actorLineColor": "#d4be98", "signalColor": "#7daea3", "signalTextColor": "#d4be98", "labelBoxBkgColor": "#32302f", "labelBoxBorderColor": "#d4be98", "labelTextColor": "#d4be98", "loopTextColor": "#d4be98"}}%%
sequenceDiagram
    actor A as Admin
    participant FE as Frontend
    participant BE as Backend
    participant DB as PostgreSQL

    Note over A,DB: 1. INICIAR CANCELACION
    A->>FE: Abre detalle del pedido #123
    FE->>BE: GET /api/admin/orders/123
    BE->>DB: Select order + payments + reservas
    DB-->>BE: Pedido PAGADO, reserva activa
    BE-->>FE: Detalle con boton Cancelar

    Note over A,DB: 2. CONFIRMAR
    A->>FE: Ingresa motivo, confirma
    FE->>BE: PATCH /api/admin/orders/123/cancel

    BE->>BE: @Transactional
    BE->>DB: Validar estado cancelable
    DB-->>BE: PAGADO -> OK
    Note over BE,DB: Para MVP: el pago aprobado NO se modifica.
    Note over BE,DB: El reembolso es gestion manual fuera del sistema.
    BE->>DB: UPDATE order -> CANCELADO
    BE->>DB: UPDATE reservation -> LIBERADA
    BE->>DB: UPDATE branch_stock (reservado -=)
    BE->>DB: INSERT stock_movement
    BE->>DB: INSERT audit_log (reembolso_pendiente)
    BE-->>FE: Pedido cancelado
    FE-->>A: Stock liberado. Gestionar reembolso manualmente.
```

---

## Reglas de cancelacion

| Estado actual | Se puede cancelar? | Accion sobre stock |
|---|---|---|
| PENDIENTE_PAGO | Si | No hay reserva |
| PAGADO | Si | Liberar reserva |
| EN_PREPARACION | Si | Liberar reserva |
| LISTO_PARA_RETIRAR | Solo admin | Liberar reserva |
| EN_REPARTO | Solo admin | Liberar reserva |
| ENTREGADO | No | Stock ya descontado |
| CANCELADO | No | Ya lo esta |

---

> [!seealso] Notas relacionadas
> - [[03 - Dominio/Reglas de Stock]]
> - [[03 - Dominio/Maquina de Estados]]
> - Volver a [[08 - Procesos/_Index]]
