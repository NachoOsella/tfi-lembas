---
title: "Requisitos Funcionales"
tags:
  - requisitos
  - funcionales
  - rf
---

# Requisitos Funcionales (MVP)

---

## Tienda online con Mercado Pago

| ID | Requisito |
|---|---|
| RF-001 | Mostrar catalogo publico de productos activos |
| RF-002 | Buscar productos por nombre |
| RF-003 | Filtrar por categoria |
| RF-004 | Mostrar detalle de producto con precio y stock |
| RF-005 | Agregar productos al carrito local (frontend) |
| RF-006 | Crear orden online (requiere autenticacion CUSTOMER) |
| RF-007 | Crear preferencia de pago en Mercado Pago Checkout Pro |
| RF-008 | Redirigir al checkout de Mercado Pago |
| RF-009 | Recibir webhook de Mercado Pago y procesar pago |
| RF-010 | Actualizar estado de orden segun resultado del pago |
| RF-011 | Consultar estado del pedido propio |

## Autenticacion

| ID | Requisito |
|---|---|
| RF-020 | Registrar nuevo usuario (rol CUSTOMER) |
| RF-021 | Iniciar sesion con email y contrasena |

## Caja operativa

| ID | Requisito |
|---|---|
| RF-030 | Abrir caja con monto inicial de efectivo |
| RF-031 | Cerrar caja con calculo de efectivo esperado vs contado |
| RF-032 | Exigir explicacion si hay diferencia de efectivo |
| RF-033 | Registrar movimientos manuales de caja (ingreso/egreso/ajuste) |
| RF-034 | Ver caja abierta actual de la sucursal |
| RF-035 | Ver historial de cierres de caja |
| RF-036 | Ver detalle de cierre con totales por medio de pago |
| RF-037 | Impedir venta presencial sin caja abierta |
| RF-038 | Impedir dos cajas abiertas en la misma sucursal |

## Ventas presenciales (POS)

| ID | Requisito |
|---|---|
| RF-040 | Crear venta presencial (requiere caja abierta) |
| RF-041 | Buscar productos por nombre o codigo de barras |
| RF-042 | Modificar cantidades |
| RF-043 | Calcular total |
| RF-044 | Registrar pago por distintos medios (efectivo, QR, transferencia, debito, credito) |
| RF-045 | Descontar stock FEFO automaticamente al cobrar |
| RF-046 | Asociar pago a la caja abierta |

## Pagos unificados

| ID | Requisito |
|---|---|
| RF-050 | Registrar payment para toda orden (online y presencial) |
| RF-051 | Asociar payment a caja si es presencial |
| RF-052 | Consultar payments por orden |
| RF-053 | Procesar webhook de Mercado Pago con idempotencia |
| RF-054 | Rechazar pago si al confirmar no hay stock suficiente (STOCK_CONFLICT) |

## Productos y catalogo

| ID | Requisito |
|---|---|
| RF-060 | Crear, editar y desactivar productos |
| RF-061 | Asignar categoria |
| RF-062 | Registrar codigo de barras |
| RF-063 | Definir visibilidad en tienda online |
| RF-064 | Definir precio de venta |
| RF-065 | Registrar imagen principal |

## Stock

| ID | Requisito |
|---|---|
| RF-070 | Registrar stock por lote y sucursal |
| RF-071 | Registrar movimientos de stock con trazabilidad |
| RF-072 | Descontar stock por venta presencial (FEFO) |
| RF-073 | Descontar stock al confirmar pago online (FEFO) |
| RF-074 | Registrar ingresos de stock |
| RF-075 | Registrar ajustes manuales |
| RF-076 | Alertar por bajo stock y vencimientos |

## Proveedores

| ID | Requisito |
|---|---|
| RF-080 | Registrar proveedores |
| RF-081 | Asociar productos con proveedores y costo |
| RF-082 | Registrar costo manual por producto-proveedor |

## Reportes

| ID | Requisito |
|---|---|
| RF-090 | Mostrar dashboard con metricas del dia |
| RF-091 | Mostrar reporte de cierre de caja con totales por medio de pago |
| RF-092 | Mostrar recomendaciones (bajo stock, vencimientos, rotacion) |

## Auditoria

| ID | Requisito |
|---|---|
| RF-100 | Registrar cambios de precio, ajustes de stock, cancelaciones, apertura/cierre de caja, movimientos de caja |

---

> [!seealso] Notas relacionadas
> - [[No Funcionales]]
> - [[Seguridad y Roles]]
> - Volver a [[_Index]]
