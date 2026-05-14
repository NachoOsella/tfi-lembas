---
title: "Roadmap e Iteraciones"
tags:
  - planificacion
  - roadmap
  - sprints
  - jira
---

# Roadmap -- 4 Sprints (MVP)

> [!info] Planificacion Scrum para desarrollo individual. 4 sprints de 2 semanas, 12 user stories cada uno. Total: 300 story points.

---

## Balance general

| Sprint | Objetivo | Stories | Story points |
|---|---|---:|---:|
| [[#Sprint 1]] | Base tecnica, autenticacion, catalogo y tienda publica inicial | 12 | 75 |
| [[#Sprint 2]] | Stock por lotes, ordenes online, carrito local, proveedores y pagos base | 12 | 75 |
| [[#Sprint 3]] | Mercado Pago, webhook, descuento FEFO online, caja operativa y venta POS | 12 | 75 |
| [[#Sprint 4]] | Estados de pedidos, cancelacion, reportes, auditoria, seguridad y despliegue demo | 12 | 75 |

---

## Sprint 1 -- Base tecnica, autenticacion y catalogo

**Objetivo:** Construir la base tecnica y de dominio minima: entorno, migraciones core, autenticacion, roles, catalogo administrable y catalogo publico inicial.

| Story | Epica | Titulo | Prioridad | Puntos | Dependencias |
|---|---|---|---:|---:|---|
| S1-US01 | EP-00 | Preparar estructura base del proyecto | Alta | 5 | - |
| S1-US02 | EP-00 | Configurar ambiente local con Docker Compose | Alta | 5 | S1-US01 |
| S1-US03 | EP-00 | Implementar migraciones Flyway iniciales | Alta | 8 | S1-US02 |
| S1-US04 | EP-01 | Registrar clientes CUSTOMER | Alta | 5 | S1-US03 |
| S1-US05 | EP-01 | Iniciar sesion con JWT | Alta | 8 | S1-US04 |
| S1-US06 | EP-12 | Gestionar usuarios internos y roles | Alta | 8 | S1-US05 |
| S1-US07 | EP-02 | Administrar categorias de productos | Media | 5 | S1-US03 |
| S1-US08 | EP-02 | Administrar productos del catalogo | Alta | 8 | S1-US07 |
| S1-US09 | EP-02 | Publicar y pausar productos en tienda | Media | 5 | S1-US08 |
| S1-US10 | EP-03 | Mostrar catalogo publico inicial | Alta | 8 | S1-US09 |
| S1-US11 | EP-00 | Estandarizar errores y validaciones de API | Alta | 5 | S1-US04 |
| S1-US12 | EP-00 | Dejar base de testing y datos demo | Media | 5 | S1-US03 |

**Entregables:** Backend y frontend ejecutables localmente. Base PostgreSQL con migraciones core/catalog y seed minimo. Registro/login CUSTOMER e inicio de seguridad JWT. Usuarios internos con roles fijos. ABM de categorias/productos y catalogo publico publicado.

---

## Sprint 2 -- Stock, ordenes online, carrito y proveedores

**Objetivo:** Incorporar inventario real por lotes, FEFO, ordenes online pendientes de pago, carrito local, proveedores y consulta de pedidos del cliente.

| Story | Epica | Titulo | Prioridad | Puntos | Dependencias |
|---|---|---|---:|---:|---|
| S2-US01 | EP-05 | Crear modelo de stock por lotes | Alta | 8 | S1-US08 |
| S2-US02 | EP-05 | Registrar ingresos de stock | Alta | 5 | S2-US01 |
| S2-US03 | EP-05 | Implementar politica FEFO testeable | Alta | 8 | S2-US01 |
| S2-US04 | EP-05 | Registrar ajustes manuales y consumos internos | Alta | 5 | S2-US01 |
| S2-US05 | EP-03 | Mostrar disponibilidad real en tienda | Alta | 5 | S2-US01 |
| S2-US06 | EP-06 | Crear modelo de ordenes unificadas | Alta | 8 | S2-US01 |
| S2-US07 | EP-09 | Crear base de pagos unificados | Alta | 5 | S2-US06 |
| S2-US08 | EP-06 | Crear orden online pendiente de pago | Alta | 8 | S2-US05,S2-US06,S2-US07 |
| S2-US09 | EP-03 | Implementar carrito local en Angular | Alta | 5 | S2-US05 |
| S2-US10 | EP-03 | Completar flujo frontend de confirmacion online | Alta | 5 | S2-US08,S2-US09 |
| S2-US11 | EP-10 | Gestionar proveedores y costos manuales | Media | 8 | S1-US08 |
| S2-US12 | EP-03 | Consultar pedidos propios del cliente | Media | 5 | S2-US08 |

**Entregables:** Stock por lotes como fuente de verdad. Ingresos, ajustes y movimientos de stock trazables. Politica FEFO testeada. Creacion de orden online PENDING_PAYMENT sin descontar stock. Carrito local, checkout inicial y pedidos propios. Proveedores y costos manuales.

---

## Sprint 3 -- Mercado Pago, caja operativa y POS

**Objetivo:** Completar los flujos transaccionales principales: Mercado Pago, webhook, descuento FEFO online, caja operativa y venta presencial POS.

| Story | Epica | Titulo | Prioridad | Puntos | Dependencias |
|---|---|---|---:|---:|---|
| S3-US01 | EP-04 | Crear adaptador de Mercado Pago | Alta | 5 | S2-US07 |
| S3-US02 | EP-04 | Crear preferencia de pago Checkout Pro | Alta | 5 | S3-US01,S2-US08 |
| S3-US03 | EP-04 | Procesar webhook de MP con idempotencia | Alta | 8 | S3-US02 |
| S3-US04 | EP-05 | Descontar stock FEFO al aprobar pago online | Alta | 8 | S3-US03,S2-US03 |
| S3-US05 | EP-03 | Integrar checkout frontend con Mercado Pago | Alta | 5 | S3-US02 |
| S3-US06 | EP-07 | Abrir caja operativa por sucursal | Alta | 8 | S2-US07 |
| S3-US07 | EP-07 | Registrar movimientos manuales de caja | Media | 5 | S3-US06 |
| S3-US08 | EP-07 | Cerrar caja con arqueo de efectivo | Alta | 5 | S3-US06,S3-US07 |
| S3-US09 | EP-08 | Buscar productos en POS por nombre o barcode | Media | 5 | S1-US08,S2-US05 |
| S3-US10 | EP-08 | Crear venta presencial transaccional | Alta | 8 | S3-US06,S3-US09,S2-US03 |
| S3-US11 | EP-08 | Construir pantalla POS completa | Alta | 8 | S3-US10 |
| S3-US12 | EP-00 | Probar flujos criticos de pagos, caja y POS | Alta | 5 | S3-US03,S3-US04,S3-US08,S3-US10 |

**Entregables:** Checkout Pro con preferencia de pago. Webhook con idempotencia. Descuento FEFO al aprobar pago online. Caja: apertura, movimientos y cierre con arqueo de efectivo. POS transaccional con multiples medios de pago. Tests criticos de pagos/caja/stock.

---

## Sprint 4 -- Pedidos, reportes, seguridad y despliegue

**Objetivo:** Cerrar operacion backoffice y calidad final: estados de pedidos, cancelaciones, reportes, recomendaciones, auditoria, seguridad, E2E, demo y documentacion.

| Story | Epica | Titulo | Prioridad | Puntos | Dependencias |
|---|---|---|---:|---:|---|
| S4-US01 | EP-06 | Gestionar preparacion y retiro de pedidos online | Alta | 8 | S3-US04,S2-US12 |
| S4-US02 | EP-06 | Cancelar orden y revertir stock | Alta | 8 | S3-US04,S4-US01 |
| S4-US03 | EP-06 | Administrar pedidos desde backoffice | Alta | 5 | S4-US01,S4-US02 |
| S4-US04 | EP-11 | Crear dashboard operativo del dia | Media | 8 | S3-US10,S4-US03 |
| S4-US05 | EP-11 | Generar reporte de cierre de caja | Media | 5 | S3-US08,S3-US10 |
| S4-US06 | EP-11 | Implementar recomendaciones por reglas | Media | 8 | S2-US01,S4-US04 |
| S4-US07 | EP-00 | Auditar acciones criticas | Alta | 5 | S3-US08,S4-US02 |
| S4-US08 | EP-00 | Pulir seguridad, permisos y guards | Alta | 5 | S4-US03,S4-US07 |
| S4-US09 | EP-00 | Mejorar usabilidad responsive del MVP | Media | 5 | S3-US11,S4-US04 |
| S4-US10 | EP-00 | Cubrir flujos E2E principales | Alta | 8 | S4-US09,S4-US08 |
| S4-US11 | EP-00 | Preparar despliegue de demo | Alta | 5 | S4-US10 |
| S4-US12 | EP-00 | Cerrar documentacion academica y evidencia Jira | Media | 5 | S4-US11 |

**Entregables:** Preparacion, listo para retiro, entrega y cancelacion con reversa de stock. Backoffice de pedidos. Dashboard, reporte de caja y recomendaciones. Auditoria de acciones criticas. Hardening de seguridad, UX responsive, E2E y despliegue demo. Documentacion final y tablero Jira listo para tutor.

---

## Orden recomendado por sprint

1. Primero migraciones/modelo de datos y entities.
2. Despues servicios de dominio con tests unitarios.
3. Luego controllers/endpoints e integration tests.
4. Finalmente pantallas Angular, guards, interceptores y validaciones UX.
5. Cerrar cada sprint con demo integrada, no solo endpoints aislados.

---

## Riesgos y controles por sprint

| Riesgo | Control | Sprint |
|---|---|---|
| Scope creep por funcionalidades post-MVP | Mantener fuera del sprint backlog | Todos |
| Sobreventa por concurrencia POS + webhook | SELECT FOR UPDATE en stock_lots | 2-3 |
| Webhook duplicado de MP descontando dos veces | Idempotencia por provider_payment_id | 3 |
| Caja mal interpretada como control de todos los medios | Arqueo solo de efectivo; otros metodos informativos | 3-4 |
| Roles mezclados entre CUSTOMER y usuarios internos | branch_id null para CUSTOMER, rutas separadas | 1-4 |
| Reportes pesados bloqueando operacion diaria | Queries agregadas paginadas y dashboard acotado | 4 |

---

## Roadmap futuro (post-MVP)

| Funcionalidad | Prioridad |
|---|---|
| Envio a domicilio | Media |
| Promociones mas complejas | Baja |
| Multiples imagenes por producto | Baja |
| Importacion automatica de listas | Baja |
| App mobile nativa | Baja |
| Multiempresa | Baja |

---

> [!seealso] Notas relacionadas
> - [[MVP]]
> - [[Epicas]]
> - [[User Stories]]
> - Volver a [[_Index]]
