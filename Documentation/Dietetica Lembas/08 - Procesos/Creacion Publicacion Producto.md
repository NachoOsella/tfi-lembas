---
title: "Creacion y Publicacion de Producto"
tags:
  - procesos
  - productos
  - catalogo
  - publicacion
---

# Proceso: Creacion y Publicacion de Producto

> [!info] Desde que el admin crea el producto hasta que esta disponible en el catalogo online.

## Diagrama de secuencia

```mermaid
%%{init: {"theme": "base", "themeVariables": {"darkMode": true, "background": "#1d2021", "primaryColor": "#32302f", "primaryTextColor": "#d4be98", "primaryBorderColor": "#d4be98", "lineColor": "#d4be98", "secondaryColor": "#32302f", "tertiaryColor": "#1d2021", "textColor": "#d4be98", "noteBkgColor": "#32302f", "noteTextColor": "#d4be98", "noteBorderColor": "#d8a657", "actorBkg": "#32302f", "actorTextColor": "#d4be98", "actorLineColor": "#d4be98", "signalColor": "#7daea3", "signalTextColor": "#d4be98", "labelBoxBkgColor": "#32302f", "labelBoxBorderColor": "#d4be98", "labelTextColor": "#d4be98", "loopTextColor": "#d4be98"}}%%
sequenceDiagram
    actor A as Admin
    participant FE as Frontend
    participant BE as Backend
    participant DB as PostgreSQL

    Note over A,DB: 1. CREAR PRODUCTO
    A->>FE: Completa formulario
    A->>FE: Guardar
    FE->>BE: POST /api/admin/products
    BE->>BE: @Transactional
    BE->>DB: INSERT products (online_status = DRAFT)
    BE->>DB: INSERT product_tags
    BE->>DB: INSERT product_prices
    BE->>DB: INSERT branch_stock
    BE->>DB: INSERT stock_movement (INGRESO)
    BE-->>FE: Producto creado
    FE-->>A: "Producto en borrador"

    Note over A,DB: 2. PUBLICAR
    A->>FE: Click "Publicar"
    FE->>BE: PATCH /api/admin/products/{id}/publish
    BE->>DB: UPDATE online_status=PUBLISHED
    BE-->>FE: PUBLICADO
    FE-->>A: "Producto publicado!"

    Note over A,DB: 3. VISIBLE EN TIENDA
    actor C as Cliente
    C->>FE: Entra a la tienda
    FE->>BE: GET /api/public/catalog/products
    BE->>DB: SELECT online_status=PUBLISHED
    DB-->>BE: Producto nuevo incluido
    BE-->>FE: Catalogo actualizado

    Note over A,DB: 4. PAUSAR (opcional)
    A->>FE: Click "Pausar"
    FE->>BE: PATCH /api/admin/products/{id}/pause
    BE->>DB: UPDATE online_status=PAUSED
    FE-->>A: "Producto oculto de la tienda"
```

---

## Estados del producto

```text
BORRADOR -> PUBLICADO -> PAUSADO -> PUBLICADO (republicar)
                       -> DESACTIVADO
```

---

## Reglas

| Regla | Detalle |
|---|---|
| Producto puede existir sin stock | Stock inicial es opcional |
| Precio inicial requiere usuario | Queda registrado |
| BORRADOR no aparece en tienda | `online_status = DRAFT` |
| Solo admin puede publicar/pausar | Controlado por roles |

---

> [!seealso] Volver a [[08 - Procesos/_Index]]
