---
title: "Compra Online con Retiro"
tags:
  - procesos
  - compra-online
  - mercadopago
  - retiro
---

# Proceso: Compra Online con Mercado Pago Checkout Pro

> [!info] Cliente registrado, carrito local, pago con MP, retiro en sucursal.

---

## Diagrama de secuencia

```mermaid
%%{init: {"theme": "base", "themeVariables": {"darkMode": true, "background": "#1d2021", "primaryColor": "#32302f", "primaryTextColor": "#d4be98", "primaryBorderColor": "#d4be98", "lineColor": "#d4be98", "secondaryColor": "#32302f", "tertiaryColor": "#1d2021", "textColor": "#d4be98", "noteBkgColor": "#32302f", "noteTextColor": "#d4be98", "noteBorderColor": "#d8a657", "actorBkg": "#32302f", "actorTextColor": "#d4be98", "actorLineColor": "#d4be98", "signalColor": "#7daea3", "signalTextColor": "#d4be98", "labelBoxBkgColor": "#32302f", "labelBoxBorderColor": "#d4be98", "labelTextColor": "#d4be98", "loopTextColor": "#d4be98"}}%%
sequenceDiagram
    actor C as Cliente
    participant FE as Frontend
    participant BE as Backend
    participant MP as MercadoPago
    participant DB as PostgreSQL

    Note over C,DB: 0. REGISTRO Y LOGIN
    C->>FE: Se registra / Inicia sesion
    FE->>BE: POST /api/auth/register o /api/auth/login
    BE-->>FE: Token JWT (rol CUSTOMER)

    Note over C,DB: 1. CATALOGO Y CARRITO LOCAL
    C->>FE: Navega catalogo y agrega productos
    FE->>BE: GET /api/store/products
    BE->>DB: Consulta productos + stock
    DB-->>BE: Productos disponibles
    BE-->>FE: Catalogo
    FE->>FE: CartService (localStorage)

    Note over C,DB: 2. CREAR ORDEN
    C->>FE: Confirma compra
    FE->>BE: POST /api/customer/orders (con token)
    BE->>DB: Validar stock, INSERT order (type=ONLINE, status=PENDING_PAYMENT)
    BE->>DB: INSERT order_items (snapshots)
    BE->>DB: INSERT payment (provider=MERCADO_PAGO, method=CHECKOUT_PRO, status=PENDING)
    BE-->>FE: { orderId, orderNumber, total }

    Note over C,DB: 3. CHECKOUT MP
    C->>FE: Procede al pago
    FE->>BE: POST /api/customer/orders/{orderId}/checkout/mp
    BE->>MP: Crear preferencia de pago
    MP-->>BE: { initPoint, preferenceId }
    BE->>DB: UPDATE payment SET provider_preference_id, external_reference
    BE-->>FE: { initPoint }
    FE->>FE: Redirige a window.location.href = initPoint

    Note over C,DB: 4. PAGO EN MP
    C->>MP: Completa el pago en MP Checkout Pro
    MP->>FE: Redirige a splash de resultado
    FE->>BE: GET /api/customer/orders/{orderId} (consulta estado)

    Note over C,DB: 5. WEBHOOK MP
    MP->>BE: POST /api/webhooks/mercadopago
    BE->>BE: Verificar firma
    BE->>MP: Consultar estado real del pago
    MP-->>BE: Pago APPROVED
    BE->>BE: @Transactional
    BE->>DB: UPDATE payment (status=APPROVED, approved_at)
    BE->>DB: Revalidar stock (FOR UPDATE)
    BE->>DB: UPDATE stock_lots (descontar FEFO)
    BE->>DB: INSERT stock_movement (type=ONLINE_SALE)
    BE->>DB: UPDATE order (status=PAID)
    BE-->>MP: 200 OK

    Note over C,DB: 6. PREPARACION Y RETIRO
    actor E as Empleado
    E->>FE: Ve ordenes PAID
    E->>FE: Marca PREPARING → PATCH /api/admin/orders/{id}/prepare
    E->>FE: Marca READY → PATCH /api/admin/orders/{id}/ready
    C-->>E: Cliente retira
    E->>FE: Marca DELIVERED → PATCH /api/admin/orders/{id}/delivered
```

## Reglas de negocio

| Regla | Paso |
|---|---|
| Cliente debe estar registrado (rol CUSTOMER) | 0 |
| Carrito es local (localStorage) | 1 |
| Orden se crea PENDING_PAYMENT, sin descontar stock | 2 |
| Payment se crea junto con la orden (PENDING) | 2 |
| Checkout MP crea preferencia (endpoint separado) | 3 |
| Webhook verifica firma y estado real en MP | 5 |
| Si pago APPROVED: actualiza payment, descuenta FEFO, order a PAID | 5 |
| Si pago REJECTED: actualiza payment, order a PAYMENT_FAILED | 5 |
| Si pago aprobado pero sin stock: order a STOCK_CONFLICT | 5 |
| DELIVERED solo cambia estado, no descuenta stock | 6 |

---

> [!seealso] Notas relacionadas
> - [[02 - Modulos/Tienda Online]]
> - [[03 - Dominio/Reglas de Stock]]
> - [[Cancelacion Pedido]]
> - Volver a [[08 - Procesos/_Index]]
