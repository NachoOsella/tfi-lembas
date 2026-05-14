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

---

## Flujo

```text
Admin completa formulario de producto
    ↓
Admin guarda (POST /api/admin/products)
    ↓
Sistema crea producto con online_status = DRAFT
    ↓
Admin publica (PATCH /api/admin/products/{id}/status)
    ↓
Producto pasa a PUBLISHED
    ↓
Cliente ve producto en catalogo online
```

## Estados del producto

```text
DRAFT → PUBLISHED → PAUSED → PUBLISHED
                  → HIDDEN
```

## Reglas

- Producto puede existir sin stock inicial
- DRAFT no aparece en tienda online
- Solo admin puede cambiar estado online

---

> [!seealso] Volver a [[08 - Procesos/_Index]]
