---
title: "Actualizacion de Precios"
tags:
  - procesos
  - proveedores
  - precios
  - futuros
---

# Proceso: Actualizacion de Precios desde Proveedor (post-MVP)

> [!warning] Este proceso corresponde a la importacion automatica de listas de precios y queda fuera del alcance del MVP.
> En el MVP el costo se ingresa manualmente por producto-proveedor.

## Proceso simplificado para MVP

```text
Administrador abre producto desde backoffice
    ↓
Administrador va a la seccion de proveedores del producto
    ↓
Administrador ingresa nuevo costo manualmente
    ↓
Sistema muestra costo anterior como referencia
    ↓
Sistema calcula y sugiere precio de venta segun margen
    ↓
Administrador confirma
    ↓
Sistema guarda en historial y actualiza el costo
```

## Proceso completo (post-MVP, si el tiempo lo permite)

```text
Administrador sube archivo CSV/Excel con precios del proveedor
    ↓
Sistema parsea los items y los cruza con el catalogo
    ↓
Sistema compara costo anterior vs nuevo y calcula variacion
    ↓
Sistema sugiere precio de venta para cada producto
    ↓
Administrador revisa la tabla comparativa
    ↓
Administrador aprueba los cambios que correspondan
    ↓
Sistema aplica los cambios aprobados y registra historial
    ↓
Sistema actualiza costos en productos y sugiere nuevas etiquetas
```

---

> [!seealso] Volver a [[08 - Procesos/_Index]]
