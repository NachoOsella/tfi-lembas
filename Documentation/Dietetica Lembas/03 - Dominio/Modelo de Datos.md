---
title: "Modelo de Datos"
tags:
  - dominio
  - entidades
  - modelo-conceptual
---

# Modelo de Datos

> [!info] Modelo conceptual del dominio del sistema.
> La entidad `LISTA_PRECIO_PROVEEDOR` corresponde al flujo de importacion automatica (post-MVP). En el MVP el costo se ingresa manualmente por producto-proveedor.

## Separacion global vs sucursal

```text
Empresa
├── Catalogo global
│   ├── productos, categorias, marcas, etiquetas
│   └── precio base
└── Sucursales
    ├── stock fisico, reservado, lotes, vencimientos
    ├── ventas presenciales
    └── preparacion de pedidos
```

## Diagrama entidad-relacion

```mermaid
%%{init: {"theme": "base", "themeVariables": {"darkMode": true, "background": "#1d2021", "primaryColor": "#32302f", "primaryTextColor": "#d4be98", "primaryBorderColor": "#d4be98", "lineColor": "#d4be98", "secondaryColor": "#32302f", "tertiaryColor": "#1d2021", "textColor": "#d4be98", "noteBkgColor": "#32302f", "noteTextColor": "#d4be98", "noteBorderColor": "#d8a657", "actorBkg": "#32302f", "actorTextColor": "#d4be98", "actorLineColor": "#d4be98", "signalColor": "#7daea3", "signalTextColor": "#d4be98", "labelBoxBkgColor": "#32302f", "labelBoxBorderColor": "#d4be98", "labelTextColor": "#d4be98", "loopTextColor": "#d4be98"}}%%
erDiagram
    EMPRESA ||--o{ SUCURSAL : tiene
    EMPRESA ||--o{ USUARIO : administra
    EMPRESA ||--o{ PRODUCTO : posee
    SUCURSAL ||--o{ STOCK_SUCURSAL : contiene
    SUCURSAL ||--o{ VENTA_PRESENCIAL : registra
    SUCURSAL ||--o{ PEDIDO_ONLINE : prepara
    SUCURSAL ||--o{ USUARIO : asigna
    PRODUCTO ||--o{ STOCK_SUCURSAL : stock
    PRODUCTO ||--o{ LOTE_STOCK : lotes
    PRODUCTO ||--o{ MOVIMIENTO_STOCK : movimientos
    PRODUCTO }o--|| CATEGORIA : pertenece
    PRODUCTO }o--o{ ETIQUETA : clasifica
    PRODUCTO }o--o{ PROVEEDOR : abastece
    PROVEEDOR ||--o{ LISTA_PRECIO_PROVEEDOR : envia
    PROVEEDOR ||--o{ PRODUCTO_PROVEEDOR : ofrece
    PRODUCTO ||--o{ PRODUCTO_PROVEEDOR : tiene
    VENTA_PRESENCIAL ||--o{ VENTA_ITEM : contiene
    PRODUCTO ||--o{ VENTA_ITEM : vendido
    PEDIDO_ONLINE ||--o{ PEDIDO_ITEM : contiene
    PRODUCTO ||--o{ PEDIDO_ITEM : comprado
    PEDIDO_ONLINE ||--o| PAGO : paga
    PEDIDO_ONLINE ||--o| ENTREGA : entrega
    USUARIO ||--o{ MOVIMIENTO_STOCK : registra
    USUARIO ||--o{ VENTA_PRESENCIAL : realiza
```

## Entidades principales

### Empresa
Negocio dueno de la plataforma. Tiene sucursales, usuarios y productos.

### Sucursal
Ubicacion fisica. Tiene stock, empleados, ventas y prepara pedidos.

### Usuario
Persona que usa el sistema. Tiene rol (ADMIN, ENCARGADO, EMPLEADO, CLIENTE).

### Producto
Producto comercializable. Tiene categoria, marca, etiquetas, imagenes, stock por sucursal, lotes, proveedores y precio.

### PrecioProducto
Precio base global con historial. El cambio de precio impacta en sucursales y e-commerce.

### StockSucursal
Stock agregado por producto y sucursal. `stock_disponible = stock_fisico - stock_reservado`.

### LoteStock
Stock con vencimiento por sucursal. Permite FEFO, alertas y promociones por vencimiento.

### MovimientoStock
Trazabilidad de cualquier cambio de stock (INGRESO, VENTA, AJUSTE, MERMA, etc.).

### ReservaStock
Unidades apartadas para un pedido online. Estados: ACTIVA, CONFIRMADA, LIBERADA. No se implementa VENCIDA en el MVP porque la reserva se crea despues del pago aprobado.

### Promocion
Descuento global o por sucursal. En el MVP, las promociones por vencimiento se configuran manualmente a partir de alertas de lotes proximos a vencer y se aplican via `PromocionAplicacion`.

### PedidoOnline / VentaPresencial
Ordenes de compra online y ventas en sucursal. Cada una con sus items.

## Diagrama de stock y pedidos

```mermaid
%%{init: {"theme": "base", "themeVariables": {"darkMode": true, "background": "#1d2021", "primaryColor": "#32302f", "primaryTextColor": "#d4be98", "primaryBorderColor": "#d4be98", "lineColor": "#d4be98", "secondaryColor": "#32302f", "tertiaryColor": "#1d2021", "textColor": "#d4be98", "noteBkgColor": "#32302f", "noteTextColor": "#d4be98", "noteBorderColor": "#d8a657", "actorBkg": "#32302f", "actorTextColor": "#d4be98", "actorLineColor": "#d4be98", "signalColor": "#7daea3", "signalTextColor": "#d4be98", "labelBoxBkgColor": "#32302f", "labelBoxBorderColor": "#d4be98", "labelTextColor": "#d4be98", "loopTextColor": "#d4be98"}}%%
flowchart TD
    A[Producto global] --> B[Precio base global]
    A --> C[Stock por sucursal]
    C --> D[Lotes por sucursal]
    D --> E[Vencimientos]
    D --> F[Alertas de vencimiento]
    G[Venta presencial] --> C
    H[Pedido online] --> I[Sucursal seleccionada]
    I --> C
    H --> J[Reserva de stock]
    J --> K[Confirmacion o liberacion]
```

---

> [!seealso] Notas relacionadas
> - [[Reglas de Stock]]
> - [[Reglas de Precios]]
> - [[Reglas de Pedidos]]
> - [[03 - Dominio/Maquina de Estados]]
> - Volver a [[_Index]]
