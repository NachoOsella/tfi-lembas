---
title: "Requisitos Funcionales"
tags:
  - requisitos
  - funcionales
  - rf
---

# Requisitos Funcionales

> [!info] Lista completa de requisitos funcionales del sistema, organizados por area.

---

## Tienda online

| ID | Requisito | Modulo relacionado |
|---|---|---|
| RF-001 | Mostrar catalogo publico de productos activos | [[02 - Modulos/Tienda Online\|Tienda Online]] |
| RF-002 | Buscar productos por nombre | Tienda Online |
| RF-003 | Filtrar por categoria, marca, etiquetas y disponibilidad | Tienda Online |
| RF-004 | Mostrar detalle de producto | Tienda Online |
| RF-005 | Indicar disponibilidad segun sucursal seleccionada | Tienda Online |
| RF-006 | Agregar productos al carrito | Tienda Online |
| RF-007 | Modificar cantidades en el carrito | Tienda Online |
| RF-008 | Validar stock antes de confirmar pedido | Tienda Online |
| RF-009 | Seleccionar retiro en sucursal o envio | Tienda Online |
| RF-010 | Generar pedido online | Tienda Online |
| RF-011 | Consultar estado del pedido | Tienda Online |
| RF-012 | Mostrar recomendaciones de productos | Tienda Online / [[02 - Modulos/Asistente Inteligente\|IA]] |

## Pedidos y entregas

| ID | Requisito |
|---|---|
| RF-020 | Registrar pedidos online con productos, cantidades y precios |
| RF-021 | Manejar estados de pedido |
| RF-022 | Registrar datos de retiro o entrega |
| RF-023 | Asignar responsable de preparacion |
| RF-024 | Marcar pedido como listo para retirar |
| RF-025 | Marcar pedido como en reparto |
| RF-026 | Marcar pedido como entregado |
| RF-027 | Cancelar pedido segun reglas definidas |

## Pagos

| ID | Requisito |
|---|---|
| RF-030 | Registrar metodo de pago de una venta o pedido |
| RF-031 | Generar link de pago para pedidos online |
| RF-032 | Generar o registrar QR de pago para ventas presenciales |
| RF-033 | Manejar estados de pago |
| RF-034 | Asociar referencia externa al pago |
| RF-035 | Simular confirmacion de pago en ambiente de prueba |

## Empresa y sucursales

| ID | Requisito |
|---|---|
| RF-040 | Registrar datos de la empresa |
| RF-041 | Crear, editar y desactivar sucursales |
| RF-042 | Asociar empleados a sucursales |
| RF-043 | Asociar ventas y pedidos a una sucursal |
| RF-044 | Mantener stock separado por sucursal |

## Usuarios y roles

| ID | Requisito |
|---|---|
| RF-050 | Iniciar sesion |
| RF-051 | Administrar usuarios internos |
| RF-052 | Asignar roles |
| RF-053 | Restringir acciones segun permisos |
| RF-054 | Registrar auditoria de acciones criticas |

## Productos y catalogo

| ID | Requisito |
|---|---|
| RF-060 | Crear, editar y desactivar productos |
| RF-061 | Asignar categoria y marca |
| RF-062 | Asignar etiquetas comerciales |
| RF-063 | Registrar codigo de barras |
| RF-064 | Definir si un producto aparece en tienda online |
| RF-065 | Definir precio de venta |
| RF-066 | Registrar imagenes de producto (subir archivo, imagen principal, opcionales) |
| RF-067 | Definir ofertas temporales |

## Stock

| ID | Requisito | Reglas relacionadas |
|---|---|---|
| RF-070 | Registrar stock por producto y sucursal | [[03 - Dominio/Reglas de Stock\|Reglas de Stock]] |
| RF-071 | Registrar movimientos de stock | Reglas de Stock |
| RF-072 | Descontar stock por venta presencial | Reglas de Stock |
| RF-073 | Descontar o reservar stock por pedido online | Reglas de Stock |
| RF-074 | Registrar ingresos de stock | Reglas de Stock |
| RF-075 | Registrar ajustes manuales con motivo | Reglas de Stock |
| RF-076 | Registrar consumo interno | Reglas de Stock |
| RF-077 | Registrar merma, perdida o vencimiento | Reglas de Stock |
| RF-078 | Alertar por bajo stock | Reglas de Stock |
| RF-079 | Alertar por productos proximos a vencer | Reglas de Stock |

## Proveedores y precios

| ID | Requisito |
|---|---|
| RF-080 | Registrar proveedores |
| RF-081 | Asociar productos con proveedores |
| RF-082 | Registrar costo de reposicion por proveedor |
| RF-083 | Registrar costo manual por producto-proveedor |
| RF-084 | Ver historial de costos del producto (post-MVP: comparacion batch) |
| RF-085 | Sugerir precios de venta segun margen |
| RF-086 | Requerir aprobacion antes de aplicar cambios masivos de precio (post-MVP, cuando se automatice importacion) |
| RF-087 | Guardar historial de cambios de precio |

## Ventas presenciales

| ID | Requisito |
|---|---|
| RF-090 | Crear una venta presencial |
| RF-091 | Buscar productos por nombre o codigo de barras |
| RF-092 | Modificar cantidades |
| RF-093 | Modificar precio en casos autorizados |
| RF-094 | Calcular total |
| RF-095 | Registrar metodo de pago |
| RF-096 | Descontar stock automaticamente |
| RF-097 | Registrar venta asociada a empleado y sucursal |

## Etiquetas y codigo de barras

| ID | Requisito |
|---|---|
| RF-100 | Buscar productos mediante lector de codigo de barras |
| RF-101 | Imprimir etiquetas de precio |
| RF-102 | Imprimir etiquetas internas |
| RF-103 | Seleccionar productos para impresion masiva |
| RF-104 | Mostrar fecha de actualizacion de precio en la etiqueta |

## Analytics

| ID | Requisito |
|---|---|
| RF-110 | Mostrar ventas por periodo |
| RF-111 | Mostrar ventas por sucursal |
| RF-112 | Mostrar productos mas vendidos |
| RF-113 | Mostrar productos con bajo stock |
| RF-114 | Mostrar productos proximos a vencer |
| RF-115 | Mostrar pedidos por estado |
| RF-116 | Mostrar margen estimado |
| RF-117 | Mostrar sugerencias de reposicion |

## Modulo inteligente (recomendador por reglas)

> En el MVP el modulo inteligente no utiliza IA generativa ni LLM. Se basa en reglas deterministas: filtrado por etiquetas, categoria, stock disponible y datos reales del sistema. La IA generativa queda como evolucion futura.

| ID | Requisito |
|---|---|
| RF-120 | Recomendar productos en base al stock disponible |
| RF-121 | Sugerir alternativas cuando un producto no tenga stock |
| RF-122 | Recomendar combos o productos relacionados |
| RF-123 | Sugerir reposicion de productos con bajo stock |
| RF-124 | Sugerir promociones para productos proximos a vencer |
| RF-125 | Evitar recomendar productos inexistentes o sin stock |
| RF-126 | Evitar afirmaciones medicas o nutricionales no validadas |

---

> [!seealso] Notas relacionadas
> - [[No Funcionales]] -- requisitos no funcionales
> - [[Seguridad y Roles]] -- seguridad y permisos
> - Volver a [[_Index]]
