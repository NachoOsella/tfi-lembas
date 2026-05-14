---
title: "MVP - Version Minima Viable"
tags:
  - planificacion
  - mvp
  - alcance
---

# MVP - Version Minima Viable

> [!info] El MVP debe demostrar el flujo completo: tienda online con Mercado Pago Checkout Pro, caja operativa, venta POS, stock por lotes FEFO y pagos unificados.

---

## Criterio de exito

```text
CLIENTE (CUSTOMER):
  Registrarse e iniciar sesion
    ↓
  Navegar catalogo, agregar al carrito local
    ↓
  Confirmar compra → order ONLINE PENDING_PAYMENT
    ↓
  Pagar con Mercado Pago Checkout Pro
    ↓
  Recibir confirmacion → stock descontado FEFO
    ↓
  Seguir estado: PAID → PREPARING → READY → DELIVERED

EMPLEADO:
  Abrir caja con monto inicial de efectivo
    ↓
  Vender en POS con scanner y multiples medios de pago
    ↓
  Registrar movimientos manuales de caja
    ↓
  Preparar pedidos online, marcar listos y entregar
    ↓
  Cerrar caja comparando efectivo esperado vs contado

ADMIN:
  Gestionar productos, stock por lotes, proveedores y costos
    ↓
  Ver dashboard operativo del dia
    ↓
  Revisar reporte de cierre de caja
    ↓
  Ver recomendaciones automaticas por reglas
    ↓
  Cancelar ordenes con reversion de stock
```

---

## Planificacion: 4 sprints de 2 semanas

| Sprint | Objetivo | Story points |
|---|---|---:|
| [[Roadmap#Sprint 1]] | Base tecnica, autenticacion, catalogo y tienda publica inicial | 75 |
| [[Roadmap#Sprint 2]] | Stock por lotes, ordenes online, carrito local, proveedores y pagos base | 75 |
| [[Roadmap#Sprint 3]] | Mercado Pago, webhook, descuento FEFO online, caja operativa y venta POS | 75 |
| [[Roadmap#Sprint 4]] | Estados de pedidos, cancelacion, reportes, auditoria, seguridad y despliegue demo | 75 |

---

## Epics del proyecto

| Key | Epica | Descripcion |
|---|---|---|
| EP-00 | Plataforma tecnica, calidad y despliegue | Base transversal: estructura, Docker, perfiles, errores, testing, CI/CD, datos demo, documentacion y despliegue. |
| EP-01 | Autenticacion y registro | Registro CUSTOMER, login, JWT, BCrypt, sesion frontend y seguridad base. |
| EP-02 | Gestion de catalogo | Categorias, productos, precios, codigos de barras, imagen unica y estado online. |
| EP-03 | Tienda online | Catalogo publico, carrito local, checkout autenticado, consulta de pedidos propios y resultado de compra. |
| EP-04 | Mercado Pago Checkout Pro | Adaptador de pago, creacion de preferencia, webhook, idempotencia y estados de pago. |
| EP-05 | Stock | Stock por lotes y sucursal, movimientos, FEFO, ajustes, alertas de vencimiento y bajo stock. |
| EP-06 | Pedidos | Ordenes unificadas POS/ONLINE, items con snapshot, maquina de estados, preparacion, entrega y cancelacion. |
| EP-07 | Caja operativa | Apertura, caja actual, movimientos manuales, cierre, arqueo de efectivo y diferencia justificada. |
| EP-08 | Ventas presenciales POS | Venta rapida con busqueda/codigo de barras, caja abierta obligatoria, cobro y descuento FEFO. |
| EP-09 | Pagos unificados | Tabla payments comun para pagos online y presenciales, metodos, estados y trazabilidad. |
| EP-10 | Proveedores | ABM de proveedores, asociacion producto-proveedor y costo manual actual. |
| EP-11 | Reportes y recomendaciones | Dashboard, reporte de caja, recomendaciones por reglas, stock bajo, vencimientos y rotacion. |
| EP-12 | Usuarios y roles | Usuarios internos, asignacion de sucursal, RBAC por rol y visibilidad de interfaz segun permisos. |

---

## Riesgos y controles

| Riesgo | Control |
|---|---|
| Scope creep por funcionalidades post-MVP | Mantener fuera del sprint backlog salvo que el tutor los exija formalmente. |
| Sobreventa por concurrencia POS + webhook | Transacciones y bloqueo pesimista SELECT FOR UPDATE en stock_lots. |
| Webhook duplicado de MP descontando dos veces | Idempotencia por provider_payment_id y tests de duplicado. |
| Caja mal interpretada como control de todos los medios | Arqueo solo de efectivo; totales por otros metodos son informativos. |
| Roles mezclados entre CUSTOMER y usuarios internos | CUSTOMER con branch_id null y rutas customer/admin separadas. |

---

## Alcance incluido

| Area | Incluye |
|---|---|
| Autenticacion | Registro de cliente, login, JWT |
| Sucursales | 1 sucursal real, entidad preparada para mas |
| Productos | ABM, categorias, precio, marca texto, imagen unica |
| Catalogo online | Productos publicados, busqueda, detalle |
| Carrito | LocalStorage en frontend |
| Tienda online | MP Checkout Pro, webhook, solo retiro |
| Caja | Apertura, movimientos, cierre con arqueo de efectivo |
| Ventas presenciales | POS con caja, multiples medios de pago |
| Stock | Lotes con vencimiento, FEFO, movimientos |
| Pagos | Tabla unificada para online y presenciales |
| Proveedores | Registro, asociacion producto, costo manual |
| Reportes | Dashboard, reporte de caja, recomendaciones |
| Roles | ADMIN, MANAGER, EMPLOYEE, CUSTOMER |

## Alcance excluido

| Funcionalidad | Motivo |
|---|---|
| Envio a domicilio | Solo retiro en sucursal |
| Carrito persistente en BD | Se usa localStorage |
| Facturacion fiscal | Excede alcance academico |
| App mobile nativa | Duplica esfuerzo |
| Multiempresa | Unico negocio |
| Logistica externa | Post-MVP |
| Importacion automatica | Post-MVP |

---

> [!seealso] Notas relacionadas
> - [[Epicas]]
> - [[User Stories]]
> - [[Roadmap]]
> - Volver a [[_Index]]
