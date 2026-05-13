---
title: "Requisitos No Funcionales"
tags:
  - requisitos
  - no-funcionales
  - rnf
---

# Requisitos No Funcionales

> [!info] Requisitos de calidad del sistema: rendimiento, seguridad, mantenibilidad, usabilidad, escalabilidad y trazabilidad.

---

## Rendimiento

| ID | Requisito |
|---|---|
| RNF-001 | Las busquedas de productos en venta rapida deben responder idealmente en menos de 200 ms en condiciones normales |
| RNF-002 | La carga inicial del backoffice debe ser rapida y no bloquear tareas criticas |
| RNF-003 | Las tablas grandes deben usar paginacion o virtualizacion |
| RNF-004 | Las operaciones criticas de venta no deben depender de pantallas pesadas |
| RNF-005 | El frontend debe minimizar recargas completas y aprovechar lazy loading por modulo |

## Consistencia de API

| ID | Requisito |
|---|---|
| RNF-006 | Todas las respuestas de error deben seguir un formato JSON uniforme con codigo semantico, mensaje y detalles opcionales |
| RNF-007 | El frontend debe capturar errores HTTP mediante un interceptor global y mostrar notificaciones al usuario segun el tipo de error |

## Seguridad

| ID | Requisito |
|---|---|
| RNF-010 | El sistema debe requerir autenticacion para acceder al backoffice |
| RNF-011 | El sistema debe aplicar autorizacion por rol/permisos |
| RNF-012 | Las contrasenas deben almacenarse con hashing seguro |
| RNF-013 | Las acciones criticas deben quedar auditadas |
| RNF-014 | Los datos de pago no deben almacenar informacion sensible de tarjetas |

## Mantenibilidad

| ID | Requisito |
|---|---|
| RNF-020 | El backend debe organizarse en modulos funcionales |
| RNF-021 | El frontend debe organizarse por features |
| RNF-022 | La logica de negocio debe separarse de controladores y componentes visuales |
| RNF-023 | Las integraciones externas deben encapsularse mediante adaptadores |
| RNF-024 | El sistema debe contar con documentacion de arquitectura y decisiones tecnicas |

## Usabilidad

| ID | Requisito |
|---|---|
| RNF-030 | El sistema debe ser usable en pantallas moviles para tareas frecuentes |
| RNF-031 | La venta rapida debe requerir la menor cantidad de pasos posible |
| RNF-032 | Las alertas importantes deben ser visibles en el panel principal |
| RNF-033 | El checkout debe ser claro y breve |
| RNF-034 | Las interfaces deben adaptarse al rol del usuario |

## Escalabilidad moderada

| ID | Requisito |
|---|---|
| RNF-040 | El sistema debe permitir agregar nuevas sucursales sin redisenar el dominio |
| RNF-041 | El sistema debe permitir agregar nuevos modulos sin romper los existentes |
| RNF-042 | El sistema debe poder migrar integraciones simuladas a integraciones reales |

## Trazabilidad

| ID | Requisito |
|---|---|
| RNF-050 | Los movimientos de stock deben ser auditables |
| RNF-051 | Los cambios de precio deben conservar historial |
| RNF-052 | Los cambios de estado de pedidos deben conservar fecha, usuario y motivo si aplica |

---

> [!seealso] Notas relacionadas
> - [[Funcionales]] -- requisitos funcionales
> - [[Seguridad y Roles]] -- detalle de seguridad
> - [[04 - Arquitectura/Vision General]] -- stack tecnologico
> - Volver a [[_Index]]
