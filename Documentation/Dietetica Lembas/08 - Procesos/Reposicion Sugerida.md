---
title: "Reposicion Sugerida"
tags:
  - procesos
  - reposicion
  - stock-bajo
---

# Proceso: Reposicion Sugerida (Bajo Stock)

> [!info] El sistema detecta productos con bajo stock y sugiere al admin que reponer.

## Diagrama: consulta de stock bajo

```mermaid
%%{init: {"theme": "base", "themeVariables": {"darkMode": true, "background": "#1d2021", "primaryColor": "#32302f", "primaryTextColor": "#d4be98", "primaryBorderColor": "#d4be98", "lineColor": "#d4be98", "secondaryColor": "#32302f", "tertiaryColor": "#1d2021", "textColor": "#d4be98", "noteBkgColor": "#32302f", "noteTextColor": "#d4be98", "noteBorderColor": "#d8a657", "actorBkg": "#32302f", "actorTextColor": "#d4be98", "actorLineColor": "#d4be98", "signalColor": "#7daea3", "signalTextColor": "#d4be98", "labelBoxBkgColor": "#32302f", "labelBoxBorderColor": "#d4be98", "labelTextColor": "#d4be98", "loopTextColor": "#d4be98"}}%%
sequenceDiagram
    participant FE as Frontend
    participant BE as Backend
    participant DB as PostgreSQL

    Note over FE,DB: CONSULTA DE BAJO STOCK
    FE->>BE: GET /analytics/inventory?alertas=bajo-stock
    BE->>DB: Productos con stock_fisico <= stock_minimo
    DB-->>BE: Productos criticos
    BE->>DB: Ventas ultimos 30d
    DB-->>BE: Datos de rotacion
    BE->>BE: Calcular urgencia
    BE-->>FE: Ranking de productos a reponer
```

## Diagrama: proceso automatico

```mermaid
%%{init: {"theme": "base", "themeVariables": {"darkMode": true, "background": "#1d2021", "primaryColor": "#32302f", "primaryTextColor": "#d4be98", "primaryBorderColor": "#d4be98", "lineColor": "#d4be98", "secondaryColor": "#32302f", "tertiaryColor": "#1d2021", "textColor": "#d4be98", "noteBkgColor": "#32302f", "noteTextColor": "#d4be98", "noteBorderColor": "#d8a657", "actorBkg": "#32302f", "actorTextColor": "#d4be98", "actorLineColor": "#d4be98", "signalColor": "#7daea3", "signalTextColor": "#d4be98", "labelBoxBkgColor": "#32302f", "labelBoxBorderColor": "#d4be98", "labelTextColor": "#d4be98", "loopTextColor": "#d4be98"}}%%
sequenceDiagram
    participant S as Scheduler
    participant BE as Backend
    participant DB as PostgreSQL

    Note over S,DB: EJECUCION DIARIA
    S->>BE: Trigger reposicion
    BE->>DB: Productos con stock < minimo
    DB-->>BE: Lista
    BE->>DB: Ventas ultimos 30d
    DB-->>BE: Rotacion
    BE->>BE: Ranking por urgencia
    BE->>DB: INSERT alertas
```

---

## Logica de ranking

```text
urgencia = (minimo - actual) * (ventas_30d / 30)
```

---

> [!seealso] Volver a [[08 - Procesos/_Index]]
