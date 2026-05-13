---
title: "Promocion por Vencimiento"
tags:
  - procesos
  - promocion
  - vencimiento
  - oferta
---

# Proceso: Promocion por Vencimiento

> [!info] El sistema detecta lotes proximos a vencer, sugiere promocion, el admin aprueba y se muestra solo en esa sucursal.

## Diagrama: creacion de promocion

```mermaid
%%{init: {"theme": "base", "themeVariables": {"darkMode": true, "background": "#1d2021", "primaryColor": "#32302f", "primaryTextColor": "#d4be98", "primaryBorderColor": "#d4be98", "lineColor": "#d4be98", "secondaryColor": "#32302f", "tertiaryColor": "#1d2021", "textColor": "#d4be98", "noteBkgColor": "#32302f", "noteTextColor": "#d4be98", "noteBorderColor": "#d8a657", "actorBkg": "#32302f", "actorTextColor": "#d4be98", "actorLineColor": "#d4be98", "signalColor": "#7daea3", "signalTextColor": "#d4be98", "labelBoxBkgColor": "#32302f", "labelBoxBorderColor": "#d4be98", "labelTextColor": "#d4be98", "loopTextColor": "#d4be98"}}%%
sequenceDiagram
    actor A as Admin
    participant FE as Frontend
    participant BE as Backend
    participant DB as PostgreSQL

    Note over A,DB: 1. DETECTAR LOTES
    FE->>BE: GET /analytics/inventory?alertas=vencimiento
    BE->>DB: SELECT lotes con vencimiento < 30 dias
    DB-->>BE: Lotes proximos a vencer
    BE-->>FE: Alertas
    FE-->>A: "2 unidades vencen en 20 dias"

    Note over A,DB: 2. SUGERIR PROMOCION
    A->>FE: Solicita sugerencia
    FE->>BE: POST /assistant/promotion-suggestions
    BE->>DB: Select precio_base
    DB-->>BE: Precio actual
    BE->>BE: Calcular descuento sugerido
    BE-->>FE: Sugerencia: 30% off
    FE-->>A: "Sugerencia: 30% descuento"

    Note over A,DB: 3. APROBAR
    A->>FE: Ajusta descuento, confirma
    FE->>BE: POST /promotions
    BE->>DB: INSERT promotions (activa=true)
    BE->>DB: INSERT promotion_applications
    BE-->>FE: Promocion creada
```

## Diagrama: impacto en e-commerce

```mermaid
%%{init: {"theme": "base", "themeVariables": {"darkMode": true, "background": "#1d2021", "primaryColor": "#32302f", "primaryTextColor": "#d4be98", "primaryBorderColor": "#d4be98", "lineColor": "#d4be98", "secondaryColor": "#32302f", "tertiaryColor": "#1d2021", "textColor": "#d4be98", "noteBkgColor": "#32302f", "noteTextColor": "#d4be98", "noteBorderColor": "#d8a657", "actorBkg": "#32302f", "actorTextColor": "#d4be98", "actorLineColor": "#d4be98", "signalColor": "#7daea3", "signalTextColor": "#d4be98", "labelBoxBkgColor": "#32302f", "labelBoxBorderColor": "#d4be98", "labelTextColor": "#d4be98", "loopTextColor": "#d4be98"}}%%
sequenceDiagram
    actor C as Cliente Centro
    actor C2 as Cliente Nva Cordoba

    Note over C,C2: CLIENTE EN SUCURSAL CON OFERTA
    C->>FE: Entra con sucursal Centro
    FE->>BE: GET /catalog?branchId=centro
    BE->>DB: Productos + promociones activas
    DB-->>BE: Leche vegetal: 25% descuento
    BE-->>FE: precio con oferta
    FE-->>C: Ve oferta por vencimiento

    Note over C,C2: OTRA SUCURSAL NO VE OFERTA
    C2->>FE: Entra con Nueva Cordoba
    FE->>BE: GET /catalog?branchId=nva-cordoba
    BE->>DB: Promociones activas para Nva Cordoba
    DB-->>BE: Sin descuento
    BE-->>FE: Precio normal
    C2->>C2: No ve promocion
```

---

## Logica de descuento sugerido

```text
dias < 7  -> 50% off
dias < 15 -> 30% off
dias < 30 -> 20% off
sino      -> alerta temprana
```

---

> [!seealso] Volver a [[08 - Procesos/_Index]]
