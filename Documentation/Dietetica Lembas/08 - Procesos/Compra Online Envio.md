---
title: "Compra Online con Envio"
tags:
  - procesos
  - compra-online
  - envio
---

# Proceso: Compra Online con Envio a Domicilio

> [!info] Similar al flujo de retiro, pero con direccion de entrega y estado EN_REPARTO.

## Diagrama de secuencia

```mermaid
%%{init: {"theme": "base", "themeVariables": {"darkMode": true, "background": "#1d2021", "primaryColor": "#32302f", "primaryTextColor": "#d4be98", "primaryBorderColor": "#d4be98", "lineColor": "#d4be98", "secondaryColor": "#32302f", "tertiaryColor": "#1d2021", "textColor": "#d4be98", "noteBkgColor": "#32302f", "noteTextColor": "#d4be98", "noteBorderColor": "#d8a657", "actorBkg": "#32302f", "actorTextColor": "#d4be98", "actorLineColor": "#d4be98", "signalColor": "#7daea3", "signalTextColor": "#d4be98", "labelBoxBkgColor": "#32302f", "labelBoxBorderColor": "#d4be98", "labelTextColor": "#d4be98", "loopTextColor": "#d4be98"}}%%
sequenceDiagram
    actor C as Cliente
    participant FE as Frontend
    participant BE as Backend
    participant DB as PostgreSQL

    Note over C,DB: 1. SELECCION + CATALOGO
    C->>FE: Elige sucursal Centro
    FE->>BE: GET /api/public/catalog/products?branchId=1
    BE->>DB: Filtrar productos + stock
    DB-->>BE: Productos disponibles
    BE-->>FE: Catalogo filtrado

    Note over C,DB: 2. CARRITO
    C->>FE: Agrega productos
    FE->>BE: POST /api/public/orders (validacion previa)
    BE->>DB: Validar stock disponible
    DB-->>BE: Stock OK
    BE-->>FE: Carrito validado

    Note over C,DB: 3. CHECKOUT CON DIRECCION
    C->>FE: Elige ENVIO, ingresa direccion
    FE->>BE: POST /api/public/orders + delivery data
    BE->>BE: @Transactional
    BE->>DB: Re-validar stock
    BE->>DB: INSERT online_order (PENDIENTE_PAGO)
    BE->>DB: INSERT order_items
    BE->>DB: INSERT delivery (ENVIO)
    BE->>DB: INSERT payment (PENDIENTE)
    BE-->>FE: Pedido + link de pago
    FE-->>C: Link para pagar

    Note over C,DB: 4. PAGO + RESERVA
    C->>BE: Paga
    BE->>DB: Revalidar precio y stock disponible
    BE->>DB: UPDATE payment -> APROBADO
    BE->>DB: UPDATE order -> PAGADO
    BE->>DB: INSERT reservation (ACTIVA)
    BE->>DB: UPDATE branch_stock (reservado +=)
    BE-->>FE: Pago OK

    Note over C,DB: 5. PREPARACION
    actor E as Empleado
    E->>FE: Ve pedidos con envio
    E->>FE: Marca EN_PREPARACION
    FE->>BE: PATCH /api/admin/orders/{id}/prepare
    BE->>DB: UPDATE estado_pedido

    Note over C,DB: 6. REPARTO
    E->>FE: Marca EN_REPARTO
    FE->>BE: PATCH /api/admin/orders/{id}/out-for-delivery
    BE->>DB: UPDATE estado_pedido

    Note over C,DB: 7. ENTREGA
    E->>FE: Marca ENTREGADO
    FE->>BE: PATCH /api/admin/orders/{id}/delivered
    BE->>BE: @Transactional
    BE->>DB: UPDATE stock (FEFO + reserva)
    BE-->>FE: OK
```

---

## Diferencias con retiro

| Aspecto | Retiro | Envio |
|---|---|---|
| Estado intermedio | `LISTO_PARA_RETIRAR` | `EN_REPARTO` |
| Datos de entrega | Solo sucursal | Direccion + telefono |
| Quien finaliza | Cliente retira | Empleado marca entregado |

---

> [!seealso] Volver a [[08 - Procesos/_Index]]
