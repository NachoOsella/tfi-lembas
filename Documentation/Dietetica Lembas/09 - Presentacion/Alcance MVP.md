---
title: "Alcance del MVP - Lo que entra y lo que no"
tags:
  - presentacion
  - tutor
  - alcance
  - mvp
  - post-mvp
---

> [!info] Documento de referencia para la entrevista con el tutor.
> Resume de forma clara que funcionalidades estan incluidas en el MVP y cuales fueron diferidas, con su fundamento.

---

## 1. Resumen ejecutivo

```
MVP: 12 epicas | ~48 user stories | ~300 story points | 4 sprints de 2 semanas
Stack: Angular + Spring Boot + PostgreSQL + Mercado Pago
Entrega: Unico retiro en sucursal (PICKUP)
```

El MVP cubre el **flujo completo de punta a punta**: desde que un cliente navega el catalogo hasta que retira su pedido, y desde que un empleado abre caja hasta que cierra el turno con arqueo de efectivo.

Todo lo que no esta aca **no es un olvido** -- fue diferido intencionalmente por una razon documentada.

---

## 2. Lo que SI entra en el MVP

### 2.1 Autenticacion y usuarios

| Funcionalidad | Detalle |
|---|---|
| Registro de cliente (rol CUSTOMER) | Formulario con email, password, datos basicos |
| Login con JWT | Token-based, sin session server-side |
| Roles fijos | ADMIN, MANAGER, EMPLOYEE, CUSTOMER como campo directo en users |
| Perfil de cliente | Datos personales, historial de pedidos |
| Usuarios internos | CRUD de empleados con asignacion a sucursal |

### 2.2 Catalogo y productos

| Funcionalidad | Detalle |
|---|---|
| Categorias | Arbol de categorias (parent_id) |
| Productos | ABM completo con precio, marca texto, codigo de barras |
| Estados online | DRAFT -> PUBLISHED -> PAUSED -> HIDDEN |
| Imagen unica | Una imagen por producto, servida por Nginx |
| Catalogo publico | Listado con busqueda y filtro por categoria |
| Detalle de producto | Vista individual con precio, descripcion, imagen |

### 2.3 Tienda online (e-commerce)

| Funcionalidad | Detalle |
|---|---|
| Carrito local | localStorage del navegador, sin persistencia en BD |
| Compra online | Requiere registro y login (sin checkout invitado) |
| Pago con Mercado Pago | Checkout Pro con redireccion a MP |
| Webhook MP | Confirmacion asyncrona de pago con idempotencia |
| Estados de pedido | PENDING_PAYMENT -> PAID -> PREPARING -> READY -> DELIVERED |
| Consulta de pedidos | El cliente ve sus pedidos y su estado |
| Entrega | Solo retiro en sucursal (PICKUP) |

### 2.4 Stock e inventario

| Funcionalidad | Detalle |
|---|---|
| Stock por lotes | stock_lots como unica fuente de verdad |
| Vencimiento | Fecha de vencimiento por lote |
| FIFO/FEFO | Descuento automatico por fecha de vencimiento |
| Movimientos de stock | Trazabilidad completa de entrada/salida/ajuste |
| Stock por sucursal | Cada sucursal tiene su propio stock |
| Alerta de vencimiento | Productos proximos a vencer |
| Alerta de stock bajo | Productos por debajo del minimum_stock |

### 2.5 Ventas presenciales (POS)

| Funcionalidad | Detalle |
|---|---|
| Venta rapida | Busqueda por nombre o codigo de barras |
| Caja obligatoria | No se puede vender sin caja abierta |
| Multiples medios de pago | Efectivo, QR, transferencia, debito, credito |
| Descuento FEFO automatico | Stock se descuenta al cobrar (misma transaccion) |
| Scanner de codigo de barras | Input de teclado (lector como teclado) |

### 2.6 Caja operativa

| Funcionalidad | Detalle |
|---|---|
| Apertura de caja | Monto inicial de efectivo obligatorio |
| Movimientos manuales | Ingresos, egresos y ajustes durante el turno |
| Cierre de caja | Calcula efectivo esperado vs contado |
| Diferencia justificada | Si hay diferencia, se exige explicacion |
| Una caja abierta por sucursal | Control de concurrencia |

### 2.7 Pagos (unificados)

| Funcionalidad | Detalle |
|---|---|
| Tabla payments unificada | Misma tabla para pagos online y presenciales |
| Proveedores de pago | MERCADO_PAGO, MANUAL, BANK, CARD_TERMINAL |
| Metodos de pago | CHECKOUT_PRO, CASH, QR, TRANSFER, DEBIT_CARD, CREDIT_CARD |
| Estados de pago | PENDING, APPROVED, REJECTED, CANCELLED, REFUNDED, EXPIRED |

### 2.8 Proveedores

| Funcionalidad | Detalle |
|---|---|
| ABM de proveedores | Registro con datos de contacto y CUIT |
| Asociacion producto-proveedor | Un producto puede tener multiples proveedores |
| Costo manual | El empleado ingresa el costo actual del producto |
| Proveedor preferido | Flag para seleccionar el principal |

### 2.9 Reportes y recomendaciones

| Funcionalidad | Detalle |
|---|---|
| Dashboard operativo | Ventas del dia, pedidos pendientes, stock bajo |
| Reporte de cierre de caja | Totales por medio de pago, arqueo de efectivo |
| Recomendaciones por reglas | Reposicion sugerida, vencimientos cercanos, rotacion |
| Sin IA generativa | Recomendador rule-based, sin OpenAI ni LLMs |

### 2.10 Auditoria

| Funcionalidad | Detalle |
|---|---|
| Audit logs | Registro de acciones criticas (cambios de precio, ajustes de stock, cancelaciones) |
| Trazabilidad | stock_movements con order_id como FK directo |

---

## 3. Lo que NO entra en el MVP (y por que)

### 3.1 Envio a domicilio

| Item | Estado |
|---|---|
| Delivery | **EXCLUIDO** |
| Integracion con Rappi / PedidosYa | **EXCLUIDO** |
| Zonas de cobertura | **EXCLUIDO** |
| Costo de envio variable | **EXCLUIDO** |
| Seguimiento de repartidor | **EXCLUIDO** |

**Fundamento**: La unica modalidad de entrega en el MVP es retiro en sucursal. Agregar delivery implica gestion de direcciones, calculo de costos de envio, y potencialmente integracion con APIs de logistica externa. No aporta valor academico adicional al flujo principal.

**Como se preparo la arquitectura**: La columna `orders.fulfillment_type` ya existe como VARCHAR con valor `PICKUP`. Cuando se quiera agregar delivery, solo se agrega `DELIVERY` al CHECK. La tabla `customer_addresses` esta identificada como post-MVP en el modelo de datos.

- **Si es solo reparto manual** (empleado lleva el pedido): esfuerzo moderado, no requiere cambio estructural.
- **Si es con API externa** (Rappi, PedidosYa): se integra detras de un `ShippingGateway` siguiendo el mismo patron que `PaymentGateway`.

---

### 3.2 Facturacion fiscal

| Item | Estado |
|---|---|
| Factura electronica (AFIP/ARCA) | **EXCLUIDO** |
| Comprobantes fiscales | **EXCLUIDO** |
| Responsabilidad IVA | **EXCLUIDO** |
| Punto de venta | **EXCLUIDO** |

**Fundamento**: Excede el alcance legal y tecnico del TPF. Requiere conocimiento profundo de la normativa fiscal argentina, homologacion de software, y manejo de claves fiscales. No es relevante para demostrar las competencias tecnicas del trabajo.

---

### 3.3 App mobile nativa

| Item | Estado |
|---|---|
| App Android | **EXCLUIDO** |
| App iOS | **EXCLUIDO** |
| PWA | **EXCLUIDO** |

**Fundamento**: Duplicaria el esfuerzo de desarrollo (otro frontend, otro equipo de herramientas). La tienda online funciona en el navegador del movil con responsive design. Una app nativa no agregaria valor academico significativo.

---

### 3.4 Multiempresa / Multi-local

| Item | Estado |
|---|---|
| Soporte para multiples negocios | **EXCLUIDO** |
| Entidad companies | **EXCLUIDO** |
| Configuracion por empresa | **EXCLUIDO** |

**Fundamento**: Dietetica Lembas es un unico negocio con sucursales. La tabla `branches` ya soporta multiples sucursales, pero la capa de "empresa" (multi-negocio) queda fuera. ADR-024 documenta esta decision.

---

### 3.5 Importacion automatica de listas de precios

| Item | Estado |
|---|---|
| Parseo de Excel/PDF de proveedores | **EXCLUIDO** |
| Actualizacion masiva de costos | **EXCLUIDO** |
| Aprobacion humana de cambios masivos | **EXCLUIDO** |

**Fundamento**: El costo de proveedor se ingresa manualmente. La importacion requiere manejo de formatos variables (cada proveedor tiene su propio formato), logica de mapeo de productos, y un flujo de aprobacion de cambios (ADR-009). El esfuerzo de implementar esto para el primer proveedor es alto y no demuestra conceptos nuevos.

---

### 3.6 Carrito persistente en BD

| Item | Estado |
|---|---|
| Carrito entre sesiones | **EXCLUIDO** |
| Carrito en distintos dispositivos | **EXCLUIDO** |

**Fundamento**: El carrito se maneja con localStorage del navegador (ADR-041). Suficiente para el caso de uso del MVP. Un carrito persistente requeriria tablas `carts` y `cart_items`, sincronizacion con el backend, y manejo de sesiones.

---

### 3.7 Notificaciones

| Item | Estado |
|---|---|
| Email de confirmacion de pedido | **EXCLUIDO** |
| WhatsApp / SMS | **EXCLUIDO** |
| Notificaciones push | **EXCLUIDO** |

**Fundamento**: El seguimiento del pedido se hace consultando el estado en la web. Agregar notificaciones requiere integracion con servicios externos (email API, Twilio, etc.) que no son centrales para la logica de negocio.

---

### 3.8 IA Generativa / Chatbot

| Item | Estado |
|---|---|
| Chatbot con LLM | **EXCLUIDO** |
| OpenAI / Gemini integration | **EXCLUIDO** |
| Procesamiento de lenguaje natural | **EXCLUIDO** |

**Fundamento**: El modulo de recomendaciones usa reglas del sistema (productos cerca de vencer, bajo stock, alta rotacion). ADR-008 documenta: "Evita invenciones y respuestas incorrectas. Sin IA generativa."

---

### 3.9 Cupones y descuentos complejos

| Item | Estado |
|---|---|
| Cupones de descuento | **EXCLUIDO** |
| Descuentos por categoria | **EXCLUIDO** |
| Descuentos por lote | **EXCLUIDO** |
| Programas de fidelidad | **EXCLUIDO** |

**Fundamento**: Las promociones en el MVP son solo por producto (descuento directo). Cupones y descuentos complejos requieren logica de validacion, combinacion de reglas, y seguimiento de uso. Postergado (ADR-017).

---

### 3.10 Varios menores

| Item | Estado | Fundamento |
|---|---|---|
| Checkout invitado (sin registro) | **EXCLUIDO** | ADR-039: compra online requiere registro y login |
| Multi-idioma | **EXCLUIDO** | Unico mercado local |
| Multi-moneda | **EXCLUIDO** | Solo ARS |
| Transferencias entre sucursales | **EXCLUIDO** | Una sucursal en MVP |
| Impresion de etiquetas persistente | **EXCLUIDO** | ADR-027: PDF bajo demanda, sin persistencia |
| Marcas como tabla separada | **EXCLUIDO** | brand_name como texto en producto |
| Multiples imagenes por producto | **EXCLUIDO** | image_url unica |

---

## 4. Tabla rapida de referencia (una ojeada)

| Modulo | Funcionalidad | MVP | Post-MVP |
|---|---|---|---|
| Auth | Registro y login | SI | -- |
| Auth | Checkout invitado | -- | Futuro |
| Catalogo | ABM productos, categorias | SI | -- |
| Catalogo | Multiples imagenes | -- | Futuro |
| Tienda | Carrito local | SI | -- |
| Tienda | Carrito persistente en BD | -- | Futuro |
| Tienda | Pago con MP Checkout Pro | SI | -- |
| Tienda | Retiro en sucursal | SI | -- |
| Tienda | Envio a domicilio | -- | Futuro |
| Tienda | Notificaciones email/WhatsApp | -- | Futuro |
| Stock | Lotes con vencimiento, FEFO | SI | -- |
| Stock | Transferencias entre sucursales | -- | Futuro |
| POS | Venta presencial con caja | SI | -- |
| POS | Scanner codigo de barras | SI | -- |
| Caja | Apertura, movimientos, cierre | SI | -- |
| Caja | Arqueo de efectivo | SI | -- |
| Pagos | Tabla payments unificada | SI | -- |
| Pagos | Mercado Pago (online) | SI | -- |
| Pagos | Efectivo, QR, transferencia, tarjetas | SI | -- |
| Proveedores | ABM, asociacion, costo manual | SI | -- |
| Proveedores | Importacion automatica de precios | -- | Futuro |
| Reportes | Dashboard, cierre de caja, recomendaciones | SI | -- |
| Reportes | IA generativa / chatbot | -- | Futuro |
| Facturacion | Factura electronica AFIP | -- | Futuro |
| Movil | App nativa / PWA | -- | Futuro |
| Multiempresa | Soporte multi-negocio | -- | Futuro |
| Promociones | Por producto (descuento directo) | SI | -- |
| Promociones | Cupones, fidelidad, descuentos complejos | -- | Futuro |
| Auditoria | Audit logs de acciones criticas | SI | -- |

---

## 5. Que pasa si el tutor pregunta "por que no incluyeron X?"

| Pregunta tipica | Respuesta |
|---|---|
| "Por que no tiene envio?" | Decision documentada (ADR-040). El flujo de retiro demuestra exactamente las mismas competencias tecnicas. Agregar delivery es solo una tabla y un estado nuevo. |
| "Y la factura electronica?" | Excede el alcance academico. Requiere conocimiento de normativa fiscal ARCA, homologacion, y claves fiscales. No suma a la demostracion de ingenieria de software. |
| "Por que no usaron IA?" | Decision documentada (ADR-008). Preferimos un recomendador deterministico y predecible a un LLM que puede alucinar respuestas. La IA generativa no era un requisito del negocio real. |
| "App mobile?" | El frontend Angular es responsivo. Una app nativa duplicaria el esfuerzo sin valor academico adicional. |
| "Carrito no se guarda si cierro el navegador?" | localStorage lo preserva. Un carrito persistente en BD requeria sincronizacion continua y tablas extra, sin beneficio real para el caso de uso. |
| "Sin notificaciones?" | El seguimiento es pull (el cliente consulta). Las notificaciones son push y requieren infraestructura externa. No son centrales para el flujo de negocio. |

---

## 6. Mapa de crecimiento post-MVP

```
MVP (ahora)
├── Retiro en sucursal (PICKUP)
├── Stock por lotes FEFO
├── MP Checkout Pro
├── Caja operativa + POS
├── Catalogo basico
└── Reportes simples

Post-MVP 1 (corto plazo)
├── Envio a domicilio manual
├── Carrito persistente en BD
├── Cupones de descuento
├── Notificaciones basicas (email)
├── Multiples imagenes por producto
└── Transferencias entre sucursales

Post-MVP 2 (mediano plazo)
├── Integracion con Rappi / PedidosYa
├── Facturacion electronica (AFIP)
├── Importacion automatica de precios
├── App movil (PWA o nativa)
├── Checkout invitado
└── Dashboard avanzado con graficos

Post-MVP 3 (largo plazo)
├── Multiempresa
├── Multi-moneda / multi-idioma
├── IA generativa / chatbot
├── Fidelidad / puntos / membresias
└── Integracion con sistemas contables
```

---

> [!seealso] Notas relacionadas
> - [[06 - Planificacion/MVP]]
> - [[06 - Planificacion/Roadmap]]
> - [[_Meta/Decisiones Arquitectonicas]]
> - Volver a [[_Index]]
