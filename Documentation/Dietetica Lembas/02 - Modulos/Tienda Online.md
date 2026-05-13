---
title: "Modulo Tienda Online"
tags:
  - modulo
  - ecommerce
  - tienda-online
---

# Modulo Tienda Online

> [!info] Proposito: permitir que un cliente pueda consultar productos, verificar disponibilidad, comprar online y seguir su pedido.

## Funcionalidades

- Catalogo publico de productos con imagenes
- Busqueda por nombre y filtros
- Detalle de producto con galeria de imagenes
- Carrito con validacion de stock
- Checkout con seleccion de sucursal
- Retiro o envio
- Pedido con numero de seguimiento
- Link de pago integrado
- Seguimiento del estado del pedido
- Recomendaciones inteligentes

## Flujo de compra online con retiro

```mermaid
%%{init: {"theme": "base", "themeVariables": {"darkMode": true, "background": "#1d2021", "primaryColor": "#32302f", "primaryTextColor": "#d4be98", "primaryBorderColor": "#d4be98", "lineColor": "#d4be98", "secondaryColor": "#32302f", "tertiaryColor": "#1d2021", "textColor": "#d4be98", "noteBkgColor": "#32302f", "noteTextColor": "#d4be98", "noteBorderColor": "#d8a657", "actorBkg": "#32302f", "actorTextColor": "#d4be98", "actorLineColor": "#d4be98", "signalColor": "#7daea3", "signalTextColor": "#d4be98", "labelBoxBkgColor": "#32302f", "labelBoxBorderColor": "#d4be98", "labelTextColor": "#d4be98", "loopTextColor": "#d4be98"}}%%
sequenceDiagram
    actor Cliente
    participant Tienda
    participant Backend
    participant Stock
    participant Pago
    participant Backoffice

    Cliente->>Tienda: Busca productos
    Tienda->>Backend: Consulta catalogo
    Backend->>Stock: Verifica stock por sucursal
    Stock-->>Backend: Disponibilidad
    Backend-->>Tienda: Productos disponibles
    Cliente->>Tienda: Agrega al carrito
    Cliente->>Tienda: Selecciona retiro
    Tienda->>Backend: Confirma pedido
    Backend->>Stock: Valida stock (sin reservar)
    Stock-->>Backend: Stock OK
    Backend->>Pago: Genera link de pago
    Pago-->>Backend: Link generado
    Backend-->>Cliente: Pedido + link
    Cliente->>Pago: Realiza pago
    Pago-->>Backend: Pago aprobado
    Backend->>Stock: Revalida stock y crea reserva
    Backend->>Backoffice: Pedido pendiente (PAGADO)
    Backoffice->>Backend: Listo para retirar
    Backend-->>Cliente: Pedido listo
```

## Stock y disponibilidad

Antes de elegir sucursal: informacion general. Despues de elegir sucursal: stock real, precio efectivo y promociones de esa sucursal.

## Requisitos funcionales asociados

RF-001 a RF-012 en [[05 - Requisitos/Funcionales]].

---

> [!seealso] Volver a [[_Index]]
