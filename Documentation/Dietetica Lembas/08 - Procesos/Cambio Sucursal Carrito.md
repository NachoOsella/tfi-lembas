---
title: "Cambio de Sucursal"
tags:
  - procesos
  - sucursal
  - carrito
  - revalidacion
---

# Proceso: Cambio de Sucursal Durante la Compra

> [!info] Si el cliente cambia la sucursal con productos en el carrito, se revalida todo contra el stock de la nueva.

## Diagrama de secuencia

```mermaid
%%{init: {"theme": "base", "themeVariables": {"darkMode": true, "background": "#1d2021", "primaryColor": "#32302f", "primaryTextColor": "#d4be98", "primaryBorderColor": "#d4be98", "lineColor": "#d4be98", "secondaryColor": "#32302f", "tertiaryColor": "#1d2021", "textColor": "#d4be98", "noteBkgColor": "#32302f", "noteTextColor": "#d4be98", "noteBorderColor": "#d8a657", "actorBkg": "#32302f", "actorTextColor": "#d4be98", "actorLineColor": "#d4be98", "signalColor": "#7daea3", "signalTextColor": "#d4be98", "labelBoxBkgColor": "#32302f", "labelBoxBorderColor": "#d4be98", "labelTextColor": "#d4be98", "loopTextColor": "#d4be98"}}%%
sequenceDiagram
    actor C as Cliente
    participant FE as Frontend
    participant BE as Backend
    participant DB as PostgreSQL

    Note over C,DB: 1. CARRITO CON SUCURSAL ACTUAL
    C->>FE: Sucursal Centro
    C->>FE: Agrega 4 productos al carrito

    Note over C,DB: 2. CAMBIO DE SUCURSAL
    C->>FE: Cambia a Nueva Cordoba
    FE->>FE: BranchService.setSelectedBranch()
    FE->>FE: Hay items de otra sucursal

    Note over C,DB: 3. REVALIDAR
    FE->>BE: POST /api/public/orders (revalidacion de carrito)
    BE->>DB: SELECT stock de cada item
    DB-->>BE: Stock en Nueva Cordoba
    BE->>BE: Item 1: disponible 7 >= qty 2 -> OK
    BE->>BE: Item 2: disponible 0 -> SIN STOCK
    BE->>BE: Item 3: disponible 1 < qty 2 -> INSUFICIENTE
    BE-->>FE: Resultados por item

    Note over C,DB: 4. AJUSTAR CARRITO
    FE->>FE: Eliminar items sin stock
    FE->>FE: Reducir cantidad si insuficiente
    FE-->>C: Carrito ajustado
```

---

## Reglas de revalidacion

| Situacion | Accion |
|---|---|
| Stock suficiente | Se mantiene igual |
| Sin stock | Se elimina con aviso |
| Stock insuficiente | Se reduce al maximo disponible |
| Carrito vacio | No hay nada que revalidar |

---

> [!seealso] Volver a [[08 - Procesos/_Index]]
