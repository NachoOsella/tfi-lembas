---
title: "Estrategia de Testing"
tags:
  - referencia
  - testing
  - calidad
---

# Estrategia de Testing

> [!info] Estrategia de pruebas para backend, frontend y flujos end-to-end.

---

## Backend

### Unit tests

Probar reglas de negocio de forma aislada:

- Reglas de stock (disponible, reservado, liberacion)
- Calculo de precios sugeridos
- Cambios de estado de pedidos
- Validacion de stock contra pedidos
- Recomendaciones basicas del asistente

### Integration tests

Probar flujos completos con base de datos real:

- Crear pedido sin reservar stock
- Aprobar pago, revalidar stock y crear reserva
- Venta presencial y descuento de stock
- Ingreso manual de costo por producto-proveedor
- Actualizacion de precio sugerido al cambiar costo

### Tests con base real

Usar **Testcontainers** con PostgreSQL para flujos criticos que requieren transaccionalidad real.

---

## Frontend

Probar:

- Componentes criticos (carrito, checkout, venta rapida)
- Guards de rutas (autenticacion, roles)
- Formularios de producto
- Validaciones de stock en checkout

---

## End-to-end

Flujos candidatos para pruebas E2E:

| Flujo | Descripcion |
|---|---|
| Cliente realiza pedido con retiro | Catalogo -> Carrito -> Checkout -> Pago -> Tracking |
| Cliente realiza pedido con envio | Catalogo -> Carrito -> Checkout con direccion -> Pago |
| Empleado registra venta presencial | Buscar producto -> Agregar al ticket -> Cobrar -> Stock se descuenta |
| Administrador actualiza precio e imprime etiqueta | Cambiar precio -> Aprobar -> Imprimir etiqueta |
| Administrador revisa bajo stock | Dashboard muestra alertas -> Sugerencia de reposicion |

---

## Casos criticos de prueba

| Caso | Resultado esperado |
|---|---|
| Comprar mas unidades que stock disponible | El sistema rechaza la operacion |
| Cancelar pedido con stock reservado | El stock se libera |
| Modificar costo de producto sin permiso de admin | El sistema registra el cambio pero requiere aprobacion para aplicarlo al precio de venta |
| Empleado intenta ver analytics global | Acceso denegado |
| IA no encuentra productos disponibles | Responde sin inventar productos |
| Dos usuarios compran el ultimo producto simultaneamente | Solo un pedido se confirma (transaccion) |
| Cambio de sucursal en el carrito | El carrito se revalida contra nueva sucursal |

---

> [!seealso] Notas relacionadas
> - Volver a [[_Index]]
