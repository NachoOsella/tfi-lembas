---
title: "Decisiones Arquitectonicas"
tags:
  - meta
  - arquitectura
  - adr
---

# Decisiones Arquitectonicas (ADR)

> [!info] Registro de decisiones arquitectonicas del proyecto. Las ADR marcadas con [MVP] aplican especificamente al alcance de la version minima viable.

| ID | Decision | Motivo |
|---|---|---|
| ADR-001 | Usar monolito modular | Menor complejidad para una tesis individual. Evita overhead de microservicios. |
| ADR-002 | Separar tienda online y backoffice en el frontend | Tienen usuarios y flujos distintos (clientes vs empleados). |
| ADR-003 | Usar PostgreSQL | Modelo relacional fuerte para stock, ventas y pedidos. Soporta transacciones. |
| ADR-004 | Usar adaptadores para pagos | Permite mock en test e integracion real MP sin rediseno. |
| ADR-005 | No implementar facturacion fiscal en MVP | Excede alcance legal y tecnico del TPF. |
| ADR-006 | No implementar logistica externa en MVP | Se modela entrega interna/manual. Solo retiro en sucursal. |
| ADR-007 | Implementar lector de codigo como input teclado | Es simple, realista y suficiente para MVP. |
| ADR-008 | Recomendador basado en reglas del sistema | Evita invenciones y respuestas incorrectas. Sin IA generativa. |
| ADR-009 | Revision humana obligatoria en cambios masivos de precio (post-MVP) | Cuando se implemente importacion automatica, los cambios requeriran aprobacion. |
| ADR-010 | Stock por sucursal desde el dominio base | Permite crecimiento futuro sin rediseno. |
| ADR-011 | Orden online asociada a una unica sucursal | Evita complejidad de split orders entre locales. |
| ADR-012 | E-commerce sin stock propio | El stock online se calcula desde el disponible de la sucursal. |
| ADR-013 | [MVP] Stock descontado al aprobar pago, no antes | Simplifica el flujo: no requiere tabla de reservas. Se revierte con movimiento inverso si se cancela. |
| ADR-014 | Usar Flyway como herramienta de migraciones | Simple, versionable y suficiente para el MVP. |
| ADR-015 | [MVP] Roles fijos como campo directo en usuarios | Simplifica seguridad inicial; elimina tablas roles y user_roles. |
| ADR-016 | Precio de venta directo en el producto | Elimina tabla product_prices. Historial en audit_logs. |
| ADR-017 | [MVP] Promociones solo por producto (no por categoria ni lote) | Reduce complejidad. |
| ADR-018 | Costo de proveedor manual por producto en MVP | Evita importacion y parseo de listas de precios. |
| ADR-019 | Imagenes de producto almacenadas en sistema de archivos | Simple, sin dependencias externas. Nginx sirve archivos estaticos. |
| ADR-020 | Formato uniforme de errores en API REST | Todas las respuestas de error usan ApiError. Implementado con @ControllerAdvice. |
| ADR-021 | Interceptor HTTP global en frontend para errores | Centraliza manejo de errores. |
| ADR-022 | [MVP] Orden unificada (POS y ONLINE) | Elimina duplicacion de sales y online_orders. El type distingue origen. |
| ADR-023 | [MVP] stock_lots como unica fuente de verdad de stock | SUM(quantity_available). Sin branch_product_stock ni stock_reservations. |
| ADR-024 | [MVP] Sin entidad companies (unico negocio) | Multiempresa queda como vision futura. |
| ADR-025 | [MVP] Sin tabla stock_reservations | Stock se descuenta al aprobar pago. Se revierte con CANCELLATION_RETURN. |
| ADR-026 | [MVP] Payments como tabla separada (reemplaza campos en orders) | Unifica pagos online y presenciales. Simplifica reportes y trazabilidad. Cada orden tiene payments asociados. |
| ADR-027 | [MVP] Impresion de etiquetas sin persistencia | Endpoint REST que genera PDF bajo demanda. Sin tablas label_print_jobs. |
| ADR-028 | [MVP] Recomendaciones sin persistencia de consultas | El recomendador rule-based calcula en tiempo real. |
| ADR-029 | [MVP] stock_movements usa order_id como FK directo | Simplifica consultas de trazabilidad. |
| ADR-030 | [MVP] order_items guarda snapshot completo | Garantiza reportes historicos precisos. |
| ADR-031 | [MVP] products.online_status con DRAFT/PUBLISHED/PAUSED/HIDDEN | Un unico campo cubre todos los estados de visibilidad. |
| ADR-032 | [MVP] Constraints CHECK en tablas clave | Previene datos invalidos a nivel BD. |
| ADR-033 | [RECHAZADO] branch_product_stock como cache de stock agregado | stock_lots es suficiente. |
| ADR-034 | [MVP] cash_sessions para gestion de caja presencial | Cubre apertura, movimientos, cierre y arqueo de efectivo. Toda venta presencial requiere caja abierta. |
| ADR-035 | [MVP] supplier_id y unit_cost en stock_lots | Trazabilidad de costo por lote. |
| ADR-036 | [MVP] quantity como numeric(12,3) en lugar de int | Soporta productos fraccionados. |
| ADR-037 | [MVP] products.current_cost eliminado | Costo se obtiene de supplier_products. |
| ADR-038 | [MVP] Roles: ADMIN/MANAGER/EMPLOYEE/CUSTOMER | CUSTOMER incluido desde MVP. Sin checkout invitado. |
| ADR-039 | [MVP] Compra online requiere registro y login | No existe checkout invitado. |
| ADR-040 | [MVP] Solo retiro en sucursal (sin envio) | Unica modalidad: PICKUP. Sin envio a domicilio en MVP. |
| ADR-041 | [MVP] Carrito en frontend (localStorage) | Sin persistencia en BD. |
| ADR-042 | [MVP] Mercado Pago Checkout Pro como pasarela real | La tienda online usa MP Checkout Pro. Backend crea preferencias, recibe webhooks y verifica pagos. |
| ADR-043 | [MVP] Caja operativa con arqueo de efectivo | Toda venta presencial requiere caja abierta. Cierre calcula diferencia de efectivo y exige explicacion. |
| ADR-044 | [MVP] Payments unificados (tabla payments) | Tabla comun para pagos online (MP) y presenciales (caja). Simplifica reportes y auditoria. |
| ADR-045 | [MVP] Estado STOCK_CONFLICT para excepciones de stock | Si al aprobar pago no hay stock suficiente, la orden pasa a revision manual. |
| ADR-046 | [MVP] order_type = POS para ventas presenciales | Reemplaza IN_STORE. Identifica claramente ventas de mostrador. |
| ADR-047 | [MVP] Reportes simples con queries directas | Sin tablas agregadas ni jobs programados. |

---

> Volver a [[_Index]]
