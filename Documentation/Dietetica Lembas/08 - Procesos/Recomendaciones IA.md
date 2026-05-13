---
title: "Recomendaciones IA"
tags:
  - procesos
  - ia
  - recomendaciones
---

# Proceso: Recomendaciones del Asistente Inteligente

> [!info] Desde la consulta del cliente hasta la recomendacion de productos reales y disponibles.

## Diagrama: consulta con resultados

```mermaid
%%{init: {"theme": "base", "themeVariables": {"darkMode": true, "background": "#1d2021", "primaryColor": "#32302f", "primaryTextColor": "#d4be98", "primaryBorderColor": "#d4be98", "lineColor": "#d4be98", "secondaryColor": "#32302f", "tertiaryColor": "#1d2021", "textColor": "#d4be98", "noteBkgColor": "#32302f", "noteTextColor": "#d4be98", "noteBorderColor": "#d8a657", "actorBkg": "#32302f", "actorTextColor": "#d4be98", "actorLineColor": "#d4be98", "signalColor": "#7daea3", "signalTextColor": "#d4be98", "labelBoxBkgColor": "#32302f", "labelBoxBorderColor": "#d4be98", "labelTextColor": "#d4be98", "loopTextColor": "#d4be98"}}%%
sequenceDiagram
    actor C as Cliente
    participant FE as Frontend
    participant BE as Backend
    participant DB as PostgreSQL

    Note over C,DB: 1. CONSULTA
    C->>FE: "quiero algo dulce sin azucar"
    C->>FE: Sucursal: Centro
    FE->>BE: POST /assistant/recommendations

    Note over C,DB: 2. PROCESAR
    BE->>BE: Extraer terminos: "dulce", "sin azucar"
    BE->>BE: Mapear a tags del sistema

    Note over C,DB: 3. CONSULTAR BD
    BE->>DB: SELECT productos con esos tags
    BE->>DB: + stock en sucursal Centro
    DB-->>BE: 3 productos con stock

    Note over C,DB: 4. RESPONDER
    BE->>BE: Priorizar stock > 0
    BE-->>FE: Productos recomendados
    FE-->>C: Opciones disponibles
```

## Diagrama: cuando no hay stock

```mermaid
%%{init: {"theme": "base", "themeVariables": {"darkMode": true, "background": "#1d2021", "primaryColor": "#32302f", "primaryTextColor": "#d4be98", "primaryBorderColor": "#d4be98", "lineColor": "#d4be98", "secondaryColor": "#32302f", "tertiaryColor": "#1d2021", "textColor": "#d4be98", "noteBkgColor": "#32302f", "noteTextColor": "#d4be98", "noteBorderColor": "#d8a657", "actorBkg": "#32302f", "actorTextColor": "#d4be98", "actorLineColor": "#d4be98", "signalColor": "#7daea3", "signalTextColor": "#d4be98", "labelBoxBkgColor": "#32302f", "labelBoxBorderColor": "#d4be98", "labelTextColor": "#d4be98", "loopTextColor": "#d4be98"}}%%
sequenceDiagram
    actor C as Cliente
    participant FE as Frontend
    participant BE as Backend
    participant DB as PostgreSQL

    C->>FE: "busco almendras"
    FE->>BE: POST /assistant/recommendations
    BE->>DB: Productos con tag "almendra"
    DB-->>BE: Sin stock
    BE->>DB: Buscar alternativas (misma categoria)
    DB-->>BE: Nueces y castanas con stock
    BE-->>FE: Alternativas disponibles
    FE-->>C: "Almendras sin stock. Alternativas: Nueces, Castanas"
```

---

## Logica del asistente

1. Parsear texto -> extraer terminos clave
2. Mapear terminos a filtros del sistema
3. Consultar BD con filtros + branchId
4. Si hay resultados -> ranking por stock/precio
5. Si no -> alternativas en misma categoria

---

## Reglas de seguridad

| Regla | Implementacion |
|---|---|
| No inventar productos | Solo responde con datos de la BD |
| No recomendar sin stock | Filtra stock_disponible > 0 |
| No hacer diagnosticos | No hay LLM libre, solo reglas |
| No exponer datos internos | No devuelve costos ni margenes |

---

> [!seealso] Volver a [[08 - Procesos/_Index]]
