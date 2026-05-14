---
title: "Modulo Tienda Online"
tags:
  - modulo
  - ecommerce
  - tienda-online
---

# Modulo Tienda Online (MVP)

> [!info] Tienda online con Mercado Pago Checkout Pro. Solo retiro en sucursal.

---

## Funcionalidades

- Catalogo publico de productos
- Busqueda por nombre y filtro por categoria
- Detalle de producto con imagen y precio
- Carrito local (localStorage)
- Confirmacion de compra (requiere CUSTOMER)
- Pago con Mercado Pago Checkout Pro
- Webhook de confirmacion de pago
- Pedido con numero de seguimiento
- Solo retiro en sucursal (sin envio)

## Autenticacion requerida

- Registro (rol CUSTOMER) e inicio de sesion.
- No existe checkout invitado.

## Flujo

```text
Cliente registrado → Carrito local → Order (PENDING_PAYMENT)
→ Checkout MP (preferencia) → Pago en MP
→ Webhook → Pago aprobado → Descuento FEFO
→ Preparacion → Retiro
```

---

> [!seealso] Volver a [[_Index]]
