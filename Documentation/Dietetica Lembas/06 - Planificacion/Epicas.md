---
title: "Epicas del Proyecto"
tags:
  - planificacion
  - epicas
  - backlog
  - jira
---

# Epicas del Proyecto (MVP)

> [!info] 13 epicas que cubren todo el alcance del MVP. Cada epica agrupa historias de usuario de un area funcional.

---

| Key | Epica | Descripcion | Sprint |
|---|---|---|---|
| EP-00 | Plataforma tecnica, calidad y despliegue | Base transversal del proyecto: estructura, Docker, perfiles, errores, testing, CI/CD, datos demo, documentacion y despliegue. | 1, 3, 4 |
| EP-01 | Autenticacion y registro | Registro CUSTOMER, login, JWT, BCrypt, sesion frontend y seguridad base. | 1 |
| EP-02 | Gestion de catalogo | Categorias, productos, precios, codigos de barras, imagen unica y estado online. | 1 |
| EP-03 | Tienda online | Catalogo publico, carrito local, checkout autenticado, consulta de pedidos propios y resultado de compra. | 1, 2, 3 |
| EP-04 | Mercado Pago Checkout Pro | Adaptador de pago, creacion de preferencia, webhook, idempotencia y estados de pago. | 3 |
| EP-05 | Stock | Stock por lotes y sucursal, movimientos, FEFO, ajustes, alertas de vencimiento y bajo stock. | 2, 3 |
| EP-06 | Pedidos | Ordenes unificadas POS/ONLINE, items con snapshot, maquina de estados, preparacion, entrega y cancelacion. | 2, 4 |
| EP-07 | Caja operativa | Apertura, caja actual, movimientos manuales, cierre, arqueo de efectivo y diferencia justificada. | 3 |
| EP-08 | Ventas presenciales POS | Venta rapida con busqueda/codigo de barras, caja abierta obligatoria, cobro y descuento FEFO. | 3 |
| EP-09 | Pagos unificados | Tabla payments comun para pagos online y presenciales, metodos, estados y trazabilidad. | 2, 3 |
| EP-10 | Proveedores | ABM de proveedores, asociacion producto-proveedor y costo manual actual. | 2 |
| EP-11 | Reportes y recomendaciones | Dashboard, reporte de caja, recomendaciones por reglas, stock bajo, vencimientos y rotacion. | 4 |
| EP-12 | Usuarios y roles | Usuarios internos, asignacion de sucursal, RBAC por rol y visibilidad de interfaz segun permisos. | 1 |

---

## EP-00 -- Plataforma tecnica, calidad y despliegue

Base transversal del proyecto: estructura de monolitio modular, Docker Compose, perfiles dev/test, errores uniformes, testing unitario/de integracion/E2E, datos demo semilla, documentacion tecnica y configuracion de despliegue.

**Sprints:** 1, 3, 4
**Stories:** S1-US01 (estructura), S1-US02 (Docker), S1-US03 (Flyway), S1-US11 (errores), S1-US12 (testing/demo), S3-US12 (tests criticos), S4-US07 (auditoria), S4-US08 (seguridad), S4-US09 (UX responsive), S4-US10 (E2E), S4-US11 (despliegue), S4-US12 (documentacion)

## EP-01 -- Autenticacion y registro

Registro de clientes con rol CUSTOMER, inicio de sesion con JWT, BCrypt, sesion en frontend y seguridad base. No existe checkout invitado.

**Sprint:** 1
**Stories:** S1-US04 (registro), S1-US05 (login JWT)

## EP-02 -- Gestion de catalogo

Administracion de categorias y productos desde backoffice. Codigos de barras, precio de venta, marca como texto, imagen unica, estados online (DRAFT/PUBLISHED/PAUSED/HIDDEN).

**Sprint:** 1
**Stories:** S1-US07 (categorias), S1-US08 (productos), S1-US09 (estado online)

## EP-03 -- Tienda online

Catalogo publico con busqueda y filtros, carrito local en localStorage, checkout autenticado, integracion con MP, consulta de pedidos propios del cliente y resultado de pago.

**Sprints:** 1, 2, 3
**Stories:** S1-US10 (catalogo publico), S2-US05 (disponibilidad real), S2-US09 (carrito local), S2-US10 (confirmacion frontend), S2-US12 (pedidos propios), S3-US05 (checkout MP frontend)

## EP-04 -- Mercado Pago Checkout Pro

Adaptador de pago (PaymentGateway), creacion de preferencia en MP, webhook con verificacion de firma, idempotencia por provider_payment_id, mapeo de estados MP a PaymentStatus interno.

**Sprint:** 3
**Stories:** S3-US01 (adaptador), S3-US02 (preferencia), S3-US03 (webhook)

## EP-05 -- Stock

Stock por lotes como unica fuente de verdad, movimientos de stock, politica FEFO testeable, ingresos y ajustes manuales. Alertas de vencimiento y bajo stock en reportes.

**Sprints:** 2, 3
**Stories:** S2-US01 (modelo lotes), S2-US02 (ingresos), S2-US03 (FEFO), S2-US04 (ajustes), S3-US04 (descuento FEFO online)

## EP-06 -- Pedidos

Ordenes unificadas POS/ONLINE con type, items con snapshot de producto, maquina de estados, flujo de preparacion y entrega, cancelacion con reversion de stock.

**Sprints:** 2, 4
**Stories:** S2-US06 (modelo ordenes), S2-US08 (orden online PENDING_PAYMENT), S4-US01 (gestion preparacion/retiro), S4-US02 (cancelacion), S4-US03 (backoffice pedidos)

## EP-07 -- Caja operativa

Apertura de caja con monto inicial, validacion de una sola caja abierta por sucursal, movimientos manuales, cierre con calculo de efectivo esperado vs contado y exigencia de explicacion ante diferencias.

**Sprint:** 3
**Stories:** S3-US06 (apertura), S3-US07 (movimientos manuales), S3-US08 (cierre con arqueo)

## EP-08 -- Ventas presenciales POS

Venta rapida con busqueda por nombre o codigo de barras, caja abierta obligatoria, multiples medios de pago (CASH, QR, TRANSFER, DEBIT_CARD, CREDIT_CARD), descuento FEFO transaccional.

**Sprint:** 3
**Stories:** S3-US09 (busqueda POS), S3-US10 (venta transaccional), S3-US11 (pantalla POS)

## EP-09 -- Pagos unificados

Tabla payments comun para pagos online (MERCADO_PAGO, CHECKOUT_PRO, cash_session_id null) y presenciales (MANUAL, metodo segun medio, cash_session_id obligatorio). Estados PENDING/APPROVED/REJECTED/CANCELLED/REFUNDED/EXPIRED/IN_PROCESS.

**Sprints:** 2, 3
**Stories:** S2-US07 (base payments), S3-US01 (adaptador MP), S3-US02 (preferencia), S3-US03 (webhook)

## EP-10 -- Proveedores

Registro de proveedores, asociacion producto-proveedor con SKP proveedor y costo manual actual. Sin importacion automatica en MVP.

**Sprint:** 2
**Stories:** S2-US11 (proveedores y costos)

## EP-11 -- Reportes y recomendaciones

Dashboard operativo del dia (ventas, pedidos pendientes, stock bajo, productos top), reporte de cierre de caja con totales por metodo, recomendaciones por reglas (LOW_STOCK, EXPIRING_SOON, HIGH_ROTATION, NO_MOVEMENT).

**Sprint:** 4
**Stories:** S4-US04 (dashboard), S4-US05 (reporte caja), S4-US06 (recomendaciones)

## EP-12 -- Usuarios y roles

Gestion de usuarios internos (ADMIN, MANAGER, EMPLOYEE), asignacion de sucursal obligatoria para MANAGER/EMPLOYEE, RBAC con @PreAuthorize, menu dinamico en frontend segun rol.

**Sprint:** 1
**Stories:** S1-US06 (usuarios internos y roles)

---

> [!seealso] Notas relacionadas
> - [[MVP]]
> - [[User Stories]]
> - [[Roadmap]]
> - Volver a [[_Index]]
