---
title: "Compra Online con Retiro"
tags:
  - procesos
  - compra-online
  - retiro
  - stock
---

# Proceso: Compra Online con Retiro en Sucursal

> [!info] Flujo desde que el cliente entra a la tienda hasta que retira el pedido.

## Diagrama de secuencia

```mermaid
%%{init: {"theme": "base", "themeVariables": {"darkMode": true, "background": "#1d2021", "primaryColor": "#32302f", "primaryTextColor": "#d4be98", "primaryBorderColor": "#d4be98", "lineColor": "#d4be98", "secondaryColor": "#32302f", "tertiaryColor": "#1d2021", "textColor": "#d4be98", "noteBkgColor": "#32302f", "noteTextColor": "#d4be98", "noteBorderColor": "#d8a657", "actorBkg": "#32302f", "actorTextColor": "#d4be98", "actorLineColor": "#d4be98", "signalColor": "#7daea3", "signalTextColor": "#d4be98", "labelBoxBkgColor": "#32302f", "labelBoxBorderColor": "#d4be98", "labelTextColor": "#d4be98", "loopTextColor": "#d4be98"}}%%
sequenceDiagram
    actor C as Cliente
    participant FE as Frontend
    participant BE as Backend
    participant DB as PostgreSQL

    Note over C,DB: 1. SELECCIONAR SUCURSAL
    C->>FE: Elige sucursal Centro
    FE->>BE: GET /api/public/catalog/branches/{id}
    BE->>DB: Validar branch activa
    DB-->>BE: OK
    BE-->>FE: branch data
    FE->>FE: Guarda branchId en estado

    Note over C,DB: 2. NAVEGAR CATALOGO
    C->>FE: Busca "granola"
    FE->>BE: GET /api/public/catalog/products?q=granola&branchId=1
    BE->>DB: Filtrar productos activos + stock
    DB-->>BE: Productos con stock y precio
    BE->>BE: Calcular stock_disponible
    BE-->>FE: Productos con stock disponible
    FE-->>C: Muestra resultados

    Note over C,DB: 3. AGREGAR AL CARRITO
    C->>FE: Agrega 3 unidades
    FE->>BE: POST /api/public/orders (validacion previa al confirmar)
    BE->>DB: Validar stock disponible
    DB-->>BE: Stock OK
    BE-->>FE: Item valido
    FE->>FE: CartService.addItem()

    Note over C,DB: 4. CHECKOUT
    C->>FE: Confirma pedido, elige retiro
    FE->>BE: POST /api/public/orders
    BE->>BE: @Transactional
    BE->>DB: Re-validar stock (FOR UPDATE)
    DB-->>BE: Stock sigue OK
    BE->>DB: INSERT online_order
    BE->>DB: INSERT order_items
    BE->>BE: PaymentAdapter.generateLink()
    BE->>DB: INSERT payment (PENDIENTE)
    BE-->>FE: Pedido creado + link de pago
    FE-->>C: Link para pagar

    Note over C,DB: 5. PAGO Y RESERVA
    C->>BE: Paga mediante link
    BE->>DB: Revalidar precio y stock disponible
    BE->>DB: UPDATE payment -> APROBADO
    BE->>DB: UPDATE order -> PAGADO
    BE->>DB: INSERT stock_reservation (ACTIVA)
    BE->>DB: UPDATE branch_stock (reservado +=)
    BE-->>FE: Pago confirmado

    Note over C,DB: 6. PREPARACION
    actor E as Empleado
    E->>FE: Login, ve pedidos PAGADO
    E->>FE: Marca EN_PREPARACION
    FE->>BE: PATCH /orders/{id}/prepare
    BE->>DB: UPDATE estado_pedido
    E->>FE: Marca LISTO_PARA_RETIRAR
    FE->>BE: PATCH /api/admin/orders/{id}/ready-for-pickup
    BE->>DB: UPDATE estado_pedido

    Note over C,DB: 7. RETIRO Y DESCUENTO
    C-->>E: Cliente llega a retirar
    E->>FE: Marca ENTREGADO
    FE->>BE: PATCH /api/admin/orders/{id}/delivered
    BE->>BE: @Transactional
    BE->>DB: UPDATE stock_lots (FEFO)
    BE->>DB: UPDATE branch_stock (fisico -=, reservado -=)
    BE->>DB: UPDATE reservation -> CONFIRMADA
    BE->>DB: INSERT stock_movement
    BE-->>FE: OK
    FE-->>E: Pedido entregado
```

---

## Reglas de negocio

| Regla | Paso |
|---|---|
| Cliente debe elegir sucursal antes de agregar al carrito | 1 |
| Stock disponible = stock_fisico - stock_reservado | 2 |
| Se re-valida con lock al crear pedido | 4 |
| Reserva se crea al pagar, no al crear pedido | 5 |
| Descuento definitivo se hace al entregar | 7 |
| Descuento sigue FEFO | 7 |

---

## Estados de pedido

```
PENDIENTE_PAGO -> PAGADO -> EN_PREPARACION -> LISTO_PARA_RETIRAR -> ENTREGADO
```

---

> [!seealso] Notas relacionadas
> - [[02 - Modulos/Tienda Online]]
> - [[03 - Dominio/Reglas de Stock]]
> - [[Ciclo Reserva Stock]]
> - Volver a [[08 - Procesos/_Index]]
