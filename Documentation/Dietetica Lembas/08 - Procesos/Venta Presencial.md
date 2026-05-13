---
title: "Venta Presencial"
tags:
  - procesos
  - venta-presencial
  - pos
---

# Proceso: Venta Presencial en Sucursal

> [!info] Venta en mostrador: empleado escanea productos, cobra y descuenta stock al instante.

## Diagrama de secuencia

```mermaid
%%{init: {"theme": "base", "themeVariables": {"darkMode": true, "background": "#1d2021", "primaryColor": "#32302f", "primaryTextColor": "#d4be98", "primaryBorderColor": "#d4be98", "lineColor": "#d4be98", "secondaryColor": "#32302f", "tertiaryColor": "#1d2021", "textColor": "#d4be98", "noteBkgColor": "#32302f", "noteTextColor": "#d4be98", "noteBorderColor": "#d8a657", "actorBkg": "#32302f", "actorTextColor": "#d4be98", "actorLineColor": "#d4be98", "signalColor": "#7daea3", "signalTextColor": "#d4be98", "labelBoxBkgColor": "#32302f", "labelBoxBorderColor": "#d4be98", "labelTextColor": "#d4be98", "loopTextColor": "#d4be98"}}%%
sequenceDiagram
    actor E as Empleado
    actor Cl as Cliente (fisico)
    participant FE as Frontend Backoffice
    participant BE as Backend
    participant DB as PostgreSQL

    Note over E,DB: 1. INICIO
    E->>FE: Login (branch_id del empleado)
    E->>FE: Abre venta rapida
    FE->>BE: GET /sales/new
    BE-->>FE: SaleId temporal

    Note over E,DB: 2. ESCANEAR PRODUCTOS
    E->>FE: Escanea codigo 779123456
    FE->>BE: GET /products?barcode=...&branchId=centro
    BE->>DB: Producto + stock disponible
    DB-->>BE: Producto activo con stock
    BE-->>FE: { nombre, precio, stock }
    FE-->>E: Agrega al ticket

    E->>FE: Escanea otro codigo
    FE->>BE: GET /products?barcode=...
    BE-->>FE: Segundo producto

    Note over E,DB: 3. AJUSTES
    E->>FE: Cambia cantidad a 2
    FE->>FE: Recalcula total

    Note over E,DB: 4. COBRO
    Cl-->>E: Paga con transferencia
    E->>FE: Metodo: TRANSFER
    E->>FE: Confirma venta
    FE->>BE: POST /sales

    BE->>BE: @Transactional
    BE->>DB: Validar stock final (FOR UPDATE)
    DB-->>BE: Stock OK
    BE->>DB: UPDATE branch_stock (fisico -=)
    BE->>DB: INSERT sale + sale_items
    BE->>DB: INSERT stock_movement
    BE-->>FE: Venta completada
    FE-->>E: Total: $5.500
```

---

## Diferencias con compra online

| Aspecto | Online | Presencial |
|---|---|---|
| Sucursal | Cliente elige | Del empleado logueado |
| Reserva | Se reserva al pagar | No hay reserva |
| Descuento | Al entregar (diferido) | Al instante |
| Pago | Link de pago | En el momento |
| Estados | PENDIENTE_PAGO -> ... -> ENTREGADO | COMPLETADA directo |

---

> [!seealso] Volver a [[08 - Procesos/_Index]]
