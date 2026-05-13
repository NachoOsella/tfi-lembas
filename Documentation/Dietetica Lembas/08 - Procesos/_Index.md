---
title: "Procesos Criticos del Sistema"
tags:
  - procesos
  - diagramas
  - secuencia
  - flujos
  - index
---

# Procesos Criticos del Sistema

> [!info] Diagramas de secuencia detallados para todos los procesos criticos del sistema, mostrando la interaccion entre frontend, backend y base de datos.

---

## Lista de procesos

| # | Proceso | Flujo principal |
|---|---|---|
| 01 | [[Compra Online Retiro]] | Compra online con retiro en sucursal |
| 02 | [[Compra Online Envio]] | Compra online con envio a domicilio |
| 03 | [[Venta Presencial]] | Venta en mostrador con descuento de stock |
| 04 | [[Actualizacion Precios Proveedor]] | Actualizacion manual de costo (post-MVP: importacion automatica) |
| 05 | [[Cancelacion Pedido]] | Cancelacion y liberacion de stock |
| 06 | [[Reposicion Sugerida]] | Deteccion de bajo stock y sugerencia |
| 07 | [[Promocion Vencimiento]] | Producto proximo a vencer con oferta |
| 08 | [[Recomendaciones IA]] | Asistente inteligente para cliente |
| 09 | [[Cambio Sucursal Carrito]] | Revalidacion al cambiar de sucursal |
| 10 | [[Ciclo Reserva Stock]] | Desde la reserva hasta la liberacion |
| 11 | [[Creacion Publicacion Producto]] | Crear, publicar y vender un producto |

---

## Convenciones en los diagramas

```text
Participantes:
  - Cliente / Empleado / Admin  (actor)
  - Frontend                    (Angular)
  - Backend                     (Spring Boot)
  - DB                          (PostgreSQL)
  - Pago / IA                   (adaptadores mock)

Colores semanticos:
  - Validaciones y reglas de negocio
  - Operaciones de base de datos
  - Llamadas a servicios externos
  - Estados y cambios de estado
```

---

> Volver a [[_Index]]
