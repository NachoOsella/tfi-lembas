---
title: "Modulo Backoffice"
tags:
  - modulo
  - backoffice
  - administracion
---

# Modulo Backoffice (MVP)

> [!info] Gestion de venta presencial con caja, pedidos online, stock, proveedores y reportes.

---

## Funcionalidades

### Caja operativa
- Apertura de caja con monto inicial de efectivo
- Cierre de caja con calculo de efectivo esperado vs contado
- Explicacion obligatoria ante diferencias de efectivo
- Movimientos manuales (ingreso, egreso, ajuste)
- Visualizacion de totales por medio de pago

### Ventas presenciales (POS)
- Venta rapida con escaner de codigo de barras
- Multiples medios de pago (efectivo, QR, transferencia, debito, credito)
- Requiere caja abierta
- Descuento FEFO automatico al cobrar

### Usuarios y roles
- ADMIN, MANAGER, EMPLOYEE, CUSTOMER
- ADMIN/MANAGER/EMPLOYEE pueden abrir y cerrar caja

### Productos y catalogo
- ABM de productos, categorias, precio, marca, imagen

### Stock
- Lotes con vencimiento, FEFO, alertas

### Pedidos online
- Gestion de pedidos con MP, confirmacion via webhook
- Cambio de estados, cancelacion

### Proveedores
- Registro, asociacion producto-proveedor, costo manual

### Reportes
- Dashboard, reporte de cierre de caja, recomendaciones

---

> [!seealso] Notas relacionadas
> - [[Modelo de Datos]]
> - [[Reglas de Stock]]
> - Volver a [[_Index]]
