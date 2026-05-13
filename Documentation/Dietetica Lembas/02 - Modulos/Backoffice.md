---
title: "Modulo Backoffice"
tags:
  - modulo
  - backoffice
  - erp
  - administracion
---

# Modulo Backoffice / ERP Comercial

> [!info] Proposito
> Brindar al negocio una herramienta centralizada para operar el e-commerce, la venta presencial y la administracion comercial diaria.

---

## Funcionalidades

### Gestion de empresa y sucursales

- Registro de datos de la empresa
- Creacion, edicion y desactivacion de sucursales
- Empleados asignados a sucursales
- Stock separado por sucursal

### Usuarios y roles

- Inicio de sesion
- Administracion de usuarios internos
- Asignacion de roles (Administrador, Encargado, Empleado)
- Restriccion de acciones segun permisos
- Auditoria de acciones criticas

### Productos y catalogo

- Creacion, edicion y desactivacion de productos
- Asignacion de categoria y marca
- Asignacion de etiquetas comerciales y alimentarias
- Registro de codigo de barras
- Publicacion en tienda online
- Definicion de precio de venta
- Registro de imagenes de producto (subida de archivo, imagen principal)
- Visualizacion de imagenes en listado y detalle
- Definicion de ofertas temporales

### Stock

- Registro de stock por producto y sucursal
- Movimientos de stock (ingreso, venta, ajuste, merma, consumo interno)
- Reserva de stock para pedidos online
- Alertas de bajo stock
- Alertas de productos proximos a vencer
- Gestion de lotes y vencimientos

### Proveedores y precios

- Registro de proveedores
- Asociacion producto-proveedor con costo de reposicion
- Ingreso manual del costo por producto-proveedor (con referencia al costo anterior)
- Sugerencia de precio de venta segun margen
- Historial de cambios de costo y precio
- *Post-MVP: importacion automatica de listas de precios, comparacion batch y aprobacion masiva*

### Ventas presenciales

- Creacion de venta rapida
- Busqueda por nombre o codigo de barras
- Modificacion de cantidades
- Calculo de total
- Registro de metodo de pago
- Descuento automatico de stock
- Asociacion a empleado y sucursal

### Pedidos online

- Gestion de pedidos recibidos
- Cambio de estados (preparacion, listo, reparto, entregado)
- Asignacion de responsable
- Cancelacion con liberacion de stock

### Entregas

- Registro de datos de retiro o envio
- Estado de preparacion y entrega
- Responsable de preparacion

### Etiquetas y codigo de barras

- Busqueda por lector de codigo de barras
- Impresion de etiquetas de precio
- Impresion de etiquetas internas
- Seleccion multiple para impresion masiva
- Fecha de actualizacion en la etiqueta

### Analytics y reportes

- Ventas por periodo y sucursal
- Productos mas vendidos
- Productos con bajo stock
- Productos proximos a vencer
- Pedidos por estado
- Margen estimado por producto/categoria
- Sugerencias de reposicion

### Configuracion comercial

- Configuracion de margenes
- Reglas comerciales
- Metodos de pago habilitados

---

## Flujo de venta presencial

```text
Cliente compra en sucursal
    ↓
Empleado abre venta rapida
    ↓
Escanea codigo de barras o busca producto
    ↓
Sistema agrega producto al ticket
    ↓
Empleado ajusta cantidad si corresponde
    ↓
Cliente paga efectivo/transferencia/QR
    ↓
Sistema registra venta
    ↓
Sistema descuenta stock de la sucursal
    ↓
Venta impacta en analytics
```

## Flujo de actualizacion manual de costo desde proveedor

```text
Administrador abre producto
    ↓
Administrador va a la pestana de proveedores
    ↓
Administrador ingresa nuevo costo manualmente
    ↓
Sistema muestra costo anterior como referencia
    ↓
Sistema sugiere precio de venta segun margen
    ↓
Administrador confirma cambio de costo
    ↓
Sistema registra en historial
    ↓
Sistema actualiza sugerencia de precio de venta
```

> La importacion automatica de listas de precios (CSV, comparacion batch) queda fuera del MVP y puede agregarse como mejora futura.

## Flujo de reposicion sugerida

```text
Sistema analiza ventas y stock
    ↓
Detecta productos con bajo stock o alta rotacion
    ↓
Cruza informacion con proveedor habitual
    ↓
Sugiere productos a reponer
    ↓
Administrador revisa sugerencias
    ↓
Administrador genera pedido de reposicion o tarea interna
```

---

## Consideraciones UX

- Panel principal con tareas pendientes
- Alertas de bajo stock y vencimiento
- Pedidos pendientes visibles
- Venta rapida optimizada para mostrador
- Acciones masivas para precios/etiquetas
- Interfaces distintas segun rol

---

> [!seealso] Notas relacionadas
> - [[01 - Contexto/Propuesta]] -- vision general
> - [[Tienda Online]] -- modulo e-commerce
> - [[Asistente Inteligente]] -- recomendaciones para admin
> - [[03 - Dominio/Modelo de Datos]] -- entidades del dominio
> - [[03 - Dominio/Reglas de Stock]]
> - [[03 - Dominio/Reglas de Precios]]
> - [[04 - Arquitectura/Vision General]]
> - Volver a [[_Index]]
