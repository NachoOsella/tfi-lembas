---
title: Presentacion para la duena
tags:
  - tfi
  - anexo
aliases:
  - Presentacion dueña
---

# Lembas - Propuesta funcional para la dietetica

> Documento no tecnico para explicar el valor del sistema.

## Idea principal

Lembas centraliza productos, stock, vencimientos, ventas simples, caja del turno, proveedores, listas de precios y tareas operativas en una herramienta clara y facil de usar.

## Problemas que ayuda a resolver

- Evitar errores por carga manual o datos desactualizados.
- Saber que productos hay, cuantos quedan y cuales vencen pronto.
- Registrar ventas con scanner y descontar stock automaticamente.
- Controlar caja del turno sin transformarlo en contabilidad compleja.
- Ordenar listas de precios de proveedores.
- Registrar compras simples para saber que productos ingresaron al stock.
- Comparar costos nuevos contra anteriores antes de aplicar cambios.
- Actualizar precios con revision y generar etiquetas.
- Diferenciar lo que ve la dueña de lo que ve un empleado.

## Alcance general

- Productos y stock.
- Vencimientos.
- Ventas mostrador.
- Caja.
- Proveedores.
- Listas de precios.
- Compras simples a proveedores.
- Actualizacion de precios.
- Consumo interno.
- Ofertas temporales.
- Etiquetas.
- Reportes y alertas.

## Fuera de alcance por ahora

- Facturacion fiscal o AFIP.
- Mercado Pago o tarjetas automaticas.
- Facturas de compra, impuestos, deuda o pagos a proveedores.
- Tienda online.
- Sistema contable completo.
- Multiples sucursales.
- Lectura automatica de PDF, fotos u OCR.

## Vista inicial

![Pantalla de login](../attachments/lembas-login.png)

## Roles

### Dueña / Administradora

Gestiona productos, precios, ofertas, stock, proveedores, listas, caja, empleados, reportes y auditoria.

![Home de administracion](../attachments/lembas-home-admin.png)

### Empleado

Registra ventas, consulta precios y stock, carga consumos autorizados, registra mermas permitidas y cierra su turno. No ve costos, margenes ni reportes sensibles.

![Home de empleado](../attachments/lembas-home-empleado-refinada.png)

## Flujo de venta

1. Escanear producto o buscarlo manualmente.
2. Modificar cantidad si hace falta.
3. Aplicar oferta vigente.
4. Confirmar venta.
5. El sistema descuenta stock y registra caja.

## Flujo de lista de precios

1. Proveedor envia lista por Excel o WhatsApp.
2. La lista se carga en Lembas.
3. El sistema intenta reconocer productos.
4. Se comparan costos.
5. La dueña aprueba cambios.
6. Se actualizan costos/precios seleccionados.
7. Se generan etiquetas pendientes.

## Valor esperado

- Menos planillas y anotaciones.
- Mas control de stock real.
- Menos perdidas por vencimiento.
- Ventas mas rapidas.
- Caja mas ordenada.
- Precios mas controlados.
- Mejor comparacion de proveedores.

## Compra simple a proveedor

Permite precargar una compra con proveedor, productos, cantidades y costos esperados. Cuando llega la mercaderia, se controla lo recibido, se completa la cantidad real y se cargan vencimientos/lotes manualmente si se desea. Al confirmar, el sistema ingresa stock por la cantidad recibida real. No registra facturas fiscales, impuestos ni pagos.
