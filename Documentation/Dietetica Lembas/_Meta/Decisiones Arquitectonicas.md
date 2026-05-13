---
title: "Decisiones Arquitectonicas"
tags:
  - meta
  - arquitectura
  - adr
---

# Decisiones Arquitectonicas (ADR)

> [!info] Registro de decisiones arquitectonicas iniciales del proyecto.

| ID | Decision | Motivo |
|---|---|---|
| ADR-001 | Usar monolito modular | Menor complejidad para una tesis individual. Evita overhead de microservicios. |
| ADR-002 | Separar tienda online y backoffice en el frontend | Tienen usuarios y flujos distintos (clientes vs empleados). |
| ADR-003 | Usar PostgreSQL | Modelo relacional fuerte para stock, ventas y pedidos. Soporta transacciones. |
| ADR-004 | Usar adaptadores para pagos e IA | Permite mock inicial e integracion real futura sin rediseno. |
| ADR-005 | No implementar facturacion fiscal en MVP | Excede alcance legal y tecnico del TPF. |
| ADR-006 | No implementar logistica externa en MVP | Se modela entrega interna/manual. |
| ADR-007 | Implementar lector de codigo como input teclado | Es simple, realista y suficiente para MVP. |
| ADR-008 | IA basada en datos filtrados del sistema | Evita invenciones y respuestas incorrectas. |
| ADR-009 | Revision humana obligatoria en cambios masivos de precio (post-MVP) | Cuando se implemente importacion automatica, los cambios requeriran aprobacion. En MVP el administrador aplica cambios manualmente sin flujo de aprobacion. |
| ADR-010 | Stock por sucursal desde el dominio base | Permite crecimiento futuro sin rediseno. |
| ADR-011 | Pedido online asociado a una unica sucursal | Evita complejidad de split orders entre locales. |
| ADR-012 | E-commerce sin stock propio | El stock online se calcula desde el disponible de la sucursal seleccionada. |
| ADR-013 | Reserva de stock al aprobar pago en MVP | Reduce bloqueos durante checkout y obliga a revalidar stock antes de confirmar el pago. |
| ADR-014 | Usar Flyway como herramienta de migraciones | Es simple, versionable y suficiente para el alcance del MVP. |
| ADR-015 | Roles fijos en MVP | Simplifica seguridad inicial; los permisos granulares quedan para evolucion futura. |
| ADR-016 | Precio actual separado de historial | Evita dos fuentes de verdad entre precio vigente y eventos de cambio. |
| ADR-017 | Promociones por vencimiento manuales en MVP | Mantiene alertas de vencimiento sin complejizar el checkout con descuentos por lote. |
| ADR-018 | Costo de proveedor manual por producto en MVP | Evita la complejidad de importacion, parseo y comparacion batch de listas de precios. El costo se ingresa manualmente desde el formulario del producto. |
| ADR-019 | Imagenes de producto almacenadas en sistema de archivos | Simple, sin dependencias externas. Suficiente para el volumen de una dietetica. Nginx sirve los archivos estaticos. |
| ADR-020 | Formato uniforme de errores en API REST | Todas las respuestas de error usan `ApiError` con codigo semantico, mensaje y detalles. Implementado con `@ControllerAdvice`. |
| ADR-021 | Interceptor HTTP global en frontend para errores | Centraliza el manejo de errores: 401 redirige a login, 403 muestra alerta, errores de negocio se muestran como toast. |

---

> Volver a [[_Index]]
