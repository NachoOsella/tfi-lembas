---
title: "Maquina de Estados"
tags:
  - dominio
  - estados
  - maquina-estados
---

# Maquina de Estados (MVP)

---

## Estados de orden

### ONLINE (e-commerce)

```mermaid
%%{init: {"theme": "base", "themeVariables": {"darkMode": true, "background": "#1d2021", "primaryColor": "#32302f", "primaryTextColor": "#d4be98", "primaryBorderColor": "#d4be98", "lineColor": "#d4be98", "secondaryColor": "#32302f", "tertiaryColor": "#1d2021", "textColor": "#d4be98", "noteBkgColor": "#32302f", "noteTextColor": "#d4be98", "noteBorderColor": "#d8a657", "actorBkg": "#32302f", "actorTextColor": "#d4be98", "actorLineColor": "#d4be98", "signalColor": "#7daea3", "signalTextColor": "#d4be98", "labelBoxBkgColor": "#32302f", "labelBoxBorderColor": "#d4be98", "labelTextColor": "#d4be98", "loopTextColor": "#d4be98"}}%%
stateDiagram-v2
    [*] --> PENDING_PAYMENT
    PENDING_PAYMENT --> PAID
    PENDING_PAYMENT --> PAYMENT_FAILED
    PENDING_PAYMENT --> CANCELLED
    PAID --> PREPARING
    PAID --> STOCK_CONFLICT
    PREPARING --> READY
    READY --> DELIVERED
    PAID --> CANCELLED
    PREPARING --> CANCELLED
    READY --> CANCELLED
    STOCK_CONFLICT --> CANCELLED
    STOCK_CONFLICT --> PAID
    PAYMENT_FAILED --> PENDING_PAYMENT
    CANCELLED --> [*]
    DELIVERED --> [*]
```

### POS (venta presencial)

```mermaid
%%{init: {"theme": "base", "themeVariables": {"darkMode": true, "background": "#1d2021", "primaryColor": "#32302f", "primaryTextColor": "#d4be98", "primaryBorderColor": "#d4be98", "lineColor": "#d4be98", "secondaryColor": "#32302f", "tertiaryColor": "#1d2021", "textColor": "#d4be98", "noteBkgColor": "#32302f", "noteTextColor": "#d4be98", "noteBorderColor": "#d8a657", "actorBkg": "#32302f", "actorTextColor": "#d4be98", "actorLineColor": "#d4be98", "signalColor": "#7daea3", "signalTextColor": "#d4be98", "labelBoxBkgColor": "#32302f", "labelBoxBorderColor": "#d4be98", "labelTextColor": "#d4be98", "loopTextColor": "#d4be98"}}%%
stateDiagram-v2
    [*] --> COMPLETED
    COMPLETED --> CANCELLED
    CANCELLED --> [*]
```

### Tabla de estados

| Estado | Descripcion | Aplica a |
|---|---|---|
| PENDING_PAYMENT | Orden creada, esperando pago MP | ONLINE |
| PAID | Pago confirmado, pendiente de preparacion | ONLINE |
| PREPARING | Empleado preparando productos | ONLINE |
| READY | Listo para retiro | ONLINE |
| DELIVERED | Orden completada | ONLINE / POS |
| CANCELLED | Orden cancelada | ONLINE / POS |
| PAYMENT_FAILED | Pago rechazado por MP | ONLINE |
| STOCK_CONFLICT | Pago aprobado pero sin stock suficiente | ONLINE |
| COMPLETED | Venta presencial completada | POS |

---

## Estados de pago

| Estado | Descripcion |
|---|---|
| PENDING | Pago creado, pendiente de confirmacion |
| APPROVED | Pago confirmado |
| REJECTED | Pago rechazado por el proveedor |
| CANCELLED | Pago cancelado |
| REFUNDED | Reembolsado |
| EXPIRED | Expirado sin completarse |
| IN_PROCESS | En proceso (MP) |

---

## Estados de caja

| Estado | Descripcion |
|---|---|
| OPEN | Caja abierta, acepta ventas y movimientos |
| CLOSED | Caja cerrada, no acepta operaciones |

---

## Estado de producto en tienda

```mermaid
%%{init: {"theme": "base", "themeVariables": {"darkMode": true, "background": "#1d2021", "primaryColor": "#32302f", "primaryTextColor": "#d4be98", "primaryBorderColor": "#d4be98", "lineColor": "#d4be98", "secondaryColor": "#32302f", "tertiaryColor": "#1d2021", "textColor": "#d4be98", "noteBkgColor": "#32302f", "noteTextColor": "#d4be98", "noteBorderColor": "#d8a657", "actorBkg": "#32302f", "actorTextColor": "#d4be98", "actorLineColor": "#d4be98", "signalColor": "#7daea3", "signalTextColor": "#d4be98", "labelBoxBkgColor": "#32302f", "labelBoxBorderColor": "#d4be98", "labelTextColor": "#d4be98", "loopTextColor": "#d4be98"}}%%
stateDiagram-v2
    [*] --> DRAFT
    DRAFT --> PUBLISHED
    PUBLISHED --> PAUSED
    PAUSED --> PUBLISHED
    PUBLISHED --> HIDDEN
```

| Estado | Descripcion |
|---|---|
| DRAFT | Creado, no visible |
| PUBLISHED | Visible en tienda |
| PAUSED | Temporalmente oculto |
| HIDDEN | Dado de baja |

---

> [!seealso] Notas relacionadas
> - [[Modelo de Datos]]
> - [[Reglas de Pedidos]]
> - [[Reglas de Stock]]
> - Volver a [[_Index]]
