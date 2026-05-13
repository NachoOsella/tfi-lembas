---
title: "Ciclo de Vida de la Reserva"
tags:
  - procesos
  - stock
  - reserva
  - transaccional
---

# Proceso: Ciclo de Vida de la Reserva de Stock

> [!info] Mecanismo que evita la sobreventa. La reserva pasa por ACTIVA, CONFIRMADA o LIBERADA. No se implementa VENCIDA porque en el MVP la reserva se crea despues del pago aprobado.

## Maquina de estados

```mermaid
%%{init: {"theme": "base", "themeVariables": {"darkMode": true, "background": "#1d2021", "primaryColor": "#32302f", "primaryTextColor": "#d4be98", "primaryBorderColor": "#d4be98", "lineColor": "#d4be98", "secondaryColor": "#32302f", "tertiaryColor": "#1d2021", "textColor": "#d4be98", "noteBkgColor": "#32302f", "noteTextColor": "#d4be98", "noteBorderColor": "#d8a657", "actorBkg": "#32302f", "actorTextColor": "#d4be98", "actorLineColor": "#d4be98", "signalColor": "#7daea3", "signalTextColor": "#d4be98", "labelBoxBkgColor": "#32302f", "labelBoxBorderColor": "#d4be98", "labelTextColor": "#d4be98", "loopTextColor": "#d4be98"}}%%
stateDiagram-v2
    [*] --> ACTIVA: Pago aprobado
    ACTIVA --> CONFIRMADA: Pedido entregado
    ACTIVA --> LIBERADA: Pedido cancelado
    CONFIRMADA --> [*]
    LIBERADA --> [*]
```

## Diagrama de secuencia

```mermaid
%%{init: {"theme": "base", "themeVariables": {"darkMode": true, "background": "#1d2021", "primaryColor": "#32302f", "primaryTextColor": "#d4be98", "primaryBorderColor": "#d4be98", "lineColor": "#d4be98", "secondaryColor": "#32302f", "tertiaryColor": "#1d2021", "textColor": "#d4be98", "noteBkgColor": "#32302f", "noteTextColor": "#d4be98", "noteBorderColor": "#d8a657", "actorBkg": "#32302f", "actorTextColor": "#d4be98", "actorLineColor": "#d4be98", "signalColor": "#7daea3", "signalTextColor": "#d4be98", "labelBoxBkgColor": "#32302f", "labelBoxBorderColor": "#d4be98", "labelTextColor": "#d4be98", "loopTextColor": "#d4be98"}}%%
sequenceDiagram
    participant FE as Frontend
    participant BE as Backend
    participant DB as PostgreSQL

    Note over FE,DB: FASE 1: CREAR RESERVA
    FE->>BE: Pago confirmado
    BE->>DB: INSERT reservation (ACTIVA)
    BE->>DB: UPDATE branch_stock (reservado += 3)

    Note over FE,DB: Stock: fisico=10, reservado=3, disponible=7

    Note over FE,DB: FASE 2A: CONFIRMAR (al entregar)
    FE->>BE: PATCH /orders/{id}/delivered
    BE->>DB: UPDATE reservation -> CONFIRMADA
    BE->>DB: UPDATE branch_stock (fisico -=, reservado -=)

    Note over FE,DB: Stock: fisico=7, reservado=0, disponible=7

    Note over FE,DB: FASE 2B: LIBERAR (al cancelar)
    FE->>BE: PATCH /orders/{id}/cancel
    BE->>DB: UPDATE reservation -> LIBERADA
    BE->>DB: UPDATE branch_stock (reservado -=)

    Note over FE,DB: Stock: fisico=10, reservado=0, disponible=10

    Note over FE,DB: FASE 2C: VENCER (post-MVP)
    Note over FE,DB: En MVP no hay vencimiento de reservas.
    Note over FE,DB: La reserva se crea post-pago y solo se libera al cancelar o confirmar.
```

---

## Tabla de estado del stock

| Momento | stock_fisico | stock_reservado | stock_disponible |
|---|---|---|---|
| Sin reserva | 10 | 0 | 10 |
| Reserva ACTIVA | 10 | 3 | 7 |
| CONFIRMADA (entregado) | 7 | 0 | 7 |
| LIBERADA (cancelado) | 10 | 0 | 10 |

---

> [!seealso] Notas relacionadas
> - [[03 - Dominio/Reglas de Stock]]
> - [[Cancelacion Pedido]]
> - [[Compra Online Retiro]]
> - Volver a [[08 - Procesos/_Index]]
