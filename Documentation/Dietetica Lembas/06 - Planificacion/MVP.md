---
title: "MVP - Version Minima Viable"
tags:
  - planificacion
  - mvp
  - alcance
---

# MVP - Version Minima Viable

> [!info] El MVP debe demostrar el flujo principal completo de punta a punta.

---

## Criterio de exito del MVP

> [!important] El MVP se considera exitoso si permite demostrar el siguiente flujo completo:

```text
Administrador carga productos y stock
    ↓
Cliente ve productos disponibles en tienda online
    ↓
Cliente agrega productos al carrito
    ↓
Cliente confirma pedido con retiro/envio
    ↓
Sistema genera link de pago o QR
    ↓
Pago se registra como aprobado
    ↓
Empleado prepara pedido
    ↓
Pedido cambia de estado hasta entregado
    ↓
Stock se actualiza correctamente
    ↓
Dashboard refleja ventas y stock
    ↓
Sistema sugiere reposicion/recomendaciones
```

Si este flujo funciona de punta a punta, el proyecto demuestra valor real, integracion entre modulos y complejidad suficiente para defensa.

---

## Alcance incluido en el MVP

| Area | Incluye |
|---|---|
| Autenticacion | Login y roles basicos |
| Empresa | Configuracion basica |
| Sucursales | Gestion de sucursales |
| Productos | ABM, categorias, etiquetas |
| Stock | Stock por sucursal, movimientos basicos |
| Catalogo online | Visualizacion de productos publicados |
| Carrito | Agregar/quitar productos, validacion de stock |
| Checkout | Confirmacion de pedido, seleccion de sucursal |
| Pedido online | Creacion, estados, tracking basico |
| Retiro/Envio | Datos de entrega, cambios de estado |
| Pago | Link de pago simulado, QR representativo |
| Venta presencial | Venta rapida con busqueda/scan |
| Codigo de barras | Busqueda por codigo (input teclado) |
| Proveedores | Registro basico, asociacion productos |
| Costos de proveedor | Carga manual de costo por producto-proveedor |
| Analytics | Dashboard basico con metricas principales |
| Etiquetas | Impresion simple de precios |
| IA | Recomendador acotado basado en reglas |

---

## Alcance excluido del MVP

| Funcionalidad | Motivo |
|---|---|
| Marketplace multiempresa | Complejidad excesiva |
| Franquicias complejas | Reglas avanzadas de permisos |
| Facturacion fiscal | Normativa fiscal, responsabilidad legal |
| Logistica externa (OCA, Andreani) | APIs externas, costos |
| WhatsApp automatico | API oficial, plantillas |
| OCR de facturas/listas | Complejo, no central al e-commerce |
| App mobile nativa | Duplica esfuerzo frontend |
| IA avanzada con historial individual | Privacidad, complejidad |
| Fidelizacion/puntos/cupones | No necesario para validar MVP |
| POS fiscal completo | Excede objetivo academico |
| Gestion contable completa | Transformaria el proyecto en ERP |
| Importacion automatica de listas de precios (CSV, comparacion batch) | Se ingresa el costo manualmente por producto; la automatizacion queda como mejora futura |
| Compras automaticas a proveedores | Integracion real con proveedores |
| Balanzas electronicas | Hardware especifico |

---

> [!seealso] Notas relacionadas
> - [[Epicas]] -- epics del proyecto
> - [[User Stories]] -- historias de usuario
> - [[Roadmap]] -- iteraciones y roadmap futuro
> - Volver a [[_Index]]
