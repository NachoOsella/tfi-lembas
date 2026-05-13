---
title: "Maquina de Estados"
tags:
  - dominio
  - estados
  - maquina-estados
  - workflow
---

# Maquina de Estados del Sistema

> [!info] Diagramas de estados para las entidades principales del sistema.

## Estado de pedido

> Para el MVP el pedido se crea directamente en PENDIENTE_PAGO. Se elimina RECIBIDO porque no aporta valor distinguir entre "pedido creado" y "pendiente de pago".

```mermaid
%%{init: {"theme": "base", "themeVariables": {"darkMode": true, "background": "#1d2021", "primaryColor": "#32302f", "primaryTextColor": "#d4be98", "primaryBorderColor": "#d4be98", "lineColor": "#d4be98", "secondaryColor": "#32302f", "tertiaryColor": "#1d2021", "textColor": "#d4be98", "noteBkgColor": "#32302f", "noteTextColor": "#d4be98", "noteBorderColor": "#d8a657", "actorBkg": "#32302f", "actorTextColor": "#d4be98", "actorLineColor": "#d4be98", "signalColor": "#7daea3", "signalTextColor": "#d4be98", "labelBoxBkgColor": "#32302f", "labelBoxBorderColor": "#d4be98", "labelTextColor": "#d4be98", "loopTextColor": "#d4be98"}}%%
stateDiagram-v2
    [*] --> PENDIENTE_PAGO
    PENDIENTE_PAGO --> PAGADO
    PENDIENTE_PAGO --> CANCELADO
    PAGADO --> EN_PREPARACION
    EN_PREPARACION --> LISTO_PARA_RETIRAR
    EN_PREPARACION --> EN_REPARTO
    LISTO_PARA_RETIRAR --> ENTREGADO
    EN_REPARTO --> ENTREGADO
    PAGADO --> CANCELADO
    CANCELADO --> [*]
    ENTREGADO --> [*]
```

### Estados de pedido

| Estado | Descripcion |
|---|---|
| PENDIENTE_PAGO | Pedido creado, esperando confirmacion de pago |
| PAGADO | Pago confirmado, pendiente de preparacion |
| EN_PREPARACION | Empleado preparando productos |
| LISTO_PARA_RETIRAR | Listo para retiro del cliente (solo retiro) |
| EN_REPARTO | En camino al domicilio (solo envio) |
| ENTREGADO | Pedido completado |
| CANCELADO | Pedido cancelado (libera stock si habia reserva) |

## Estado de pago

```mermaid
%%{init: {"theme": "base", "themeVariables": {"darkMode": true, "background": "#1d2021", "primaryColor": "#32302f", "primaryTextColor": "#d4be98", "primaryBorderColor": "#d4be98", "lineColor": "#d4be98", "secondaryColor": "#32302f", "tertiaryColor": "#1d2021", "textColor": "#d4be98", "noteBkgColor": "#32302f", "noteTextColor": "#d4be98", "noteBorderColor": "#d8a657", "actorBkg": "#32302f", "actorTextColor": "#d4be98", "actorLineColor": "#d4be98", "signalColor": "#7daea3", "signalTextColor": "#d4be98", "labelBoxBkgColor": "#32302f", "labelBoxBorderColor": "#d4be98", "labelTextColor": "#d4be98", "loopTextColor": "#d4be98"}}%%
stateDiagram-v2
    [*] --> PENDIENTE
    PENDIENTE --> APROBADO
    PENDIENTE --> RECHAZADO
    PENDIENTE --> CANCELADO
    APROBADO --> REEMBOLSADO
    REEMBOLSADO --> [*]
    RECHAZADO --> [*]
    CANCELADO --> [*]
```

Para MVP: PENDIENTE, APROBADO, RECHAZADO, CANCELADO.

## Estado de producto en tienda

```mermaid
%%{init: {"theme": "base", "themeVariables": {"darkMode": true, "background": "#1d2021", "primaryColor": "#32302f", "primaryTextColor": "#d4be98", "primaryBorderColor": "#d4be98", "lineColor": "#d4be98", "secondaryColor": "#32302f", "tertiaryColor": "#1d2021", "textColor": "#d4be98", "noteBkgColor": "#32302f", "noteTextColor": "#d4be98", "noteBorderColor": "#d8a657", "actorBkg": "#32302f", "actorTextColor": "#d4be98", "actorLineColor": "#d4be98", "signalColor": "#7daea3", "signalTextColor": "#d4be98", "labelBoxBkgColor": "#32302f", "labelBoxBorderColor": "#d4be98", "labelTextColor": "#d4be98", "loopTextColor": "#d4be98"}}%%
stateDiagram-v2
    [*] --> BORRADOR
    BORRADOR --> PUBLICADO
    PUBLICADO --> PAUSADO
    PAUSADO --> PUBLICADO
    PUBLICADO --> DESACTIVADO
    BORRADOR --> DESACTIVADO
```

| Estado | Descripcion |
|---|---|
| BORRADOR | Creado pero no visible |
| PUBLICADO | Visible en tienda online |
| PAUSADO | Temporalmente no visible |
| DESACTIVADO | Dado de baja |

## Estado de lista de precios (post-MVP)

> Este diagrama corresponde a la importacion automatica de listas de precios, que queda fuera del MVP.
> En el MVP el costo se ingresa manualmente por producto-proveedor sin estados intermedios.

```mermaid
%%{init: {"theme": "base", "themeVariables": {"darkMode": true, "background": "#1d2021", "primaryColor": "#32302f", "primaryTextColor": "#d4be98", "primaryBorderColor": "#d4be98", "lineColor": "#d4be98", "secondaryColor": "#32302f", "tertiaryColor": "#1d2021", "textColor": "#d4be98", "noteBkgColor": "#32302f", "noteTextColor": "#d4be98", "noteBorderColor": "#d8a657", "actorBkg": "#32302f", "actorTextColor": "#d4be98", "actorLineColor": "#d4be98", "signalColor": "#7daea3", "signalTextColor": "#d4be98", "labelBoxBkgColor": "#32302f", "labelBoxBorderColor": "#d4be98", "labelTextColor": "#d4be98", "loopTextColor": "#d4be98"}}%%
stateDiagram-v2
    [*] --> CARGADA
    CARGADA --> PROCESADA
    PROCESADA --> PENDIENTE_REVISION
    PENDIENTE_REVISION --> APROBADA
    PENDIENTE_REVISION --> DESCARTADA
    APROBADA --> APLICADA
    APLICADA --> [*]
    DESCARTADA --> [*]
```

| Estado | Descripcion |
|---|---|
| CARGADA | Lista recibida |
| PROCESADA | Items identificados y comparados |
| PENDIENTE_REVISION | Cambios listos para revision |
| APROBADA | Admin aprobo |
| APLICADA | Cambios aplicados |
| DESCARTADA | Lista descartada |

## Estado de reserva de stock

> En el MVP la reserva se crea despues de que el pago fue aprobado, por lo tanto no tiene sentido que "venza". La reserva permanece ACTIVA hasta que el pedido se entrega (CONFIRMADA) o se cancela (LIBERADA). El vencimiento de reservas es aplicable solo si se reservara antes del pago, lo cual queda fuera del MVP.

```mermaid
%%{init: {"theme": "base", "themeVariables": {"darkMode": true, "background": "#1d2021", "primaryColor": "#32302f", "primaryTextColor": "#d4be98", "primaryBorderColor": "#d4be98", "lineColor": "#d4be98", "secondaryColor": "#32302f", "tertiaryColor": "#1d2021", "textColor": "#d4be98", "noteBkgColor": "#32302f", "noteTextColor": "#d4be98", "noteBorderColor": "#d8a657", "actorBkg": "#32302f", "actorTextColor": "#d4be98", "actorLineColor": "#d4be98", "signalColor": "#7daea3", "signalTextColor": "#d4be98", "labelBoxBkgColor": "#32302f", "labelBoxBorderColor": "#d4be98", "labelTextColor": "#d4be98", "loopTextColor": "#d4be98"}}%%
stateDiagram-v2
    [*] --> ACTIVA: Pago aprobado
    ACTIVA --> CONFIRMADA: Pedido entregado
    ACTIVA --> LIBERADA: Pedido cancelado
    CONFIRMADA --> [*]
    LIBERADA --> [*]
```

| Estado | Descripcion |
|---|---|
| ACTIVA | Stock reservado, esperando entrega o cancelacion |
| CONFIRMADA | Pedido entregado, stock descontado definitivamente |
| LIBERADA | Pedido cancelado, stock liberado |

---

> [!seealso] Notas relacionadas
> - [[Modelo de Datos]]
> - [[Reglas de Pedidos]]
> - [[Reglas de Stock]]
> - Volver a [[_Index]]
