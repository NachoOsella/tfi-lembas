---
title: "User Stories"
tags:
  - planificacion
  - user-stories
  - backlog
  - jira
---

# User Stories (MVP)

> [!info] 48 user stories organizadas en 4 sprints de 2 semanas. Cada story incluye criterios de aceptacion y subtareas tecnicas. Estimacion en story points (total: 300).

---

# Sprint 1 -- Base tecnica, autenticacion y catalogo

**Objetivo:** Construir la base tecnica y de dominio minima: entorno, migraciones core, autenticacion, roles, catalogo administrable y catalogo publico inicial.
**Stories:** 12 | **Points:** 75

---

## S1-US01 -- Preparar estructura base del proyecto

**Epica:** EP-00 | **Prioridad:** Alta | **Points:** 5

**Como** desarrollador, **quiero** tener el proyecto organizado como monolito modular con frontend separado **para** poder desarrollar modulos sin mezclar responsabilidades desde el inicio.

**Criterios de aceptacion:**
- Existe estructura backend por modulos: auth, users, catalog, inventory, orders, payments, cash, suppliers, reports, audit y shared.
- Existe estructura frontend Angular por features: store, customer, admin, auth y shared.
- El proyecto levanta en local con configuracion reproducible.
- La estructura queda documentada en README tecnico.

**Subtareas sugeridas:**
- Crear repositorio/estructura con carpetas backend, frontend, infra y docs.
- Crear paquete shared para DTOs comunes, excepciones y utilidades.
- Configurar perfiles dev/test en backend y environment files en Angular.
- Agregar README con comandos de ejecucion, estructura y convenciones de ramas.
- Definir convencion de nombres para endpoints, DTOs, servicios y componentes.

---

## S1-US02 -- Configurar ambiente local con Docker Compose

**Epica:** EP-00 | **Prioridad:** Alta | **Points:** 5

**Como** desarrollador, **quiero** levantar PostgreSQL y servicios de la aplicacion con un comando **para** evitar instalaciones manuales y facilitar la demo academica.

**Criterios de aceptacion:**
- Docker Compose levanta PostgreSQL 16 con volumen persistente.
- El backend se conecta a la base usando variables de entorno.
- El frontend puede ejecutarse localmente consumiendo la API.
- Hay archivo .env.example sin credenciales reales.

**Subtareas sugeridas:**
- Crear docker-compose.yml con PostgreSQL 16 y red interna.
- Definir variables DB_HOST, DB_PORT, DB_NAME, DB_USER, DB_PASSWORD y JWT_SECRET.
- Configurar application-dev.yml y application-test.yml.
- Agregar script de reset de base local para desarrollo.
- Documentar comandos: up, down, logs, migrate y seed.

---

## S1-US03 -- Implementar migraciones Flyway iniciales

**Epica:** EP-00 | **Prioridad:** Alta | **Points:** 8

**Como** desarrollador, **quiero** versionar el esquema de base de datos desde el primer sprint **para** mantener trazabilidad y reproducibilidad del modelo.

**Criterios de aceptacion:**
- Flyway ejecuta migraciones en orden al iniciar el backend.
- Existen migraciones para branches, users, categories y products.
- Las constraints principales de roles, precios y estados online estan en base.
- Hay seed inicial minimo para operar la demo.

**Subtareas sugeridas:**
- Configurar dependencia Flyway en Spring Boot.
- Crear V1__core.sql con branches y users.
- Crear V2__catalog.sql con categories y products.
- Agregar CHECK para role ADMIN/MANAGER/EMPLOYEE/CUSTOMER.
- Agregar CHECK para online_status DRAFT/PUBLISHED/PAUSED/HIDDEN.
- Crear V10__seed_data.sql parcial con sucursal Centro, admin demo y categorias base.

---

## S1-US04 -- Registrar clientes CUSTOMER

**Epica:** EP-01 | **Prioridad:** Alta | **Points:** 5

**Como** cliente, **quiero** registrarme con email y contrasena **para** poder comprar online con una cuenta propia.

**Criterios de aceptacion:**
- POST /api/auth/register crea usuarios con rol CUSTOMER.
- El email es unico y se devuelve EMAIL_DUPLICATED ante duplicado.
- La contrasena se almacena hasheada con BCrypt.
- CUSTOMER queda con branch_id null.

**Subtareas sugeridas:**
- Crear User entity, UserRepository y Role enum.
- Crear RegisterRequest, AuthResponse y UserDto.
- Implementar AuthService.registerCustomer().
- Validar email, password, firstName, lastName y phone opcional.
- Crear tests unitarios de registro exitoso, email duplicado y password hash.

---

## S1-US05 -- Iniciar sesion con JWT

**Epica:** EP-01 | **Prioridad:** Alta | **Points:** 8

**Como** usuario, **quiero** iniciar sesion con email y contrasena **para** acceder a rutas protegidas segun mi rol.

**Criterios de aceptacion:**
- POST /api/auth/login devuelve token JWT y datos del usuario.
- Credenciales invalidas devuelven INVALID_CREDENTIALS.
- Usuarios disabled no pueden iniciar sesion.
- GET /api/auth/me devuelve usuario autenticado desde el token.

**Subtareas sugeridas:**
- Configurar Spring Security stateless y BCryptPasswordEncoder.
- Implementar JwtService para generar y validar token de 24h.
- Implementar JwtAuthenticationFilter.
- Configurar rutas publicas /api/auth/**, /api/store/**, /api/webhooks/** y /uploads/**.
- Crear AuthController con login y me.
- Agregar tests de login, token invalido y usuario deshabilitado.

---

## S1-US06 -- Gestionar usuarios internos y roles

**Epica:** EP-12 | **Prioridad:** Alta | **Points:** 8

**Como** administrador, **quiero** crear y administrar usuarios internos **para** controlar quien opera backoffice, caja, stock y pedidos.

**Criterios de aceptacion:**
- ADMIN puede listar, crear, editar, habilitar y deshabilitar usuarios internos.
- MANAGER y EMPLOYEE requieren branch_id obligatorio.
- CUSTOMER no se crea desde el ABM interno.
- Las acciones criticas de usuario quedan preparadas para auditoria.

**Subtareas sugeridas:**
- Crear UserAdminController bajo /api/admin/users.
- Implementar CreateInternalUserRequest y UpdateUserRequest.
- Validar regla branch_id segun rol.
- Aplicar @PreAuthorize para restringir ABM completo a ADMIN.
- Crear pantalla Angular de listado basico de usuarios internos.
- Agregar tests de permisos y validacion branch_id.

---

## S1-US07 -- Administrar categorias de productos

**Epica:** EP-02 | **Prioridad:** Media | **Points:** 5

**Como** administrador, **quiero** crear y editar categorias **para** organizar el catalogo y permitir filtros en la tienda.

**Criterios de aceptacion:**
- ADMIN/MANAGER pueden crear, editar y listar categorias.
- Las categorias pueden tener parent_id opcional.
- La tienda publica puede consultar categorias activas con conteo de productos cuando aplique.
- No se permite eliminar una categoria usada sin manejo controlado.

**Subtareas sugeridas:**
- Crear Category entity, repository, service y controller admin.
- Implementar DTOs CategoryRequest y CategoryDto.
- Implementar GET /api/store/categories.
- Agregar validacion de nombre obligatorio y parent existente.
- Crear componentes Angular de listado/formulario simple.
- Agregar tests de creacion, edicion y parent invalido.

---

## S1-US08 -- Administrar productos del catalogo

**Epica:** EP-02 | **Prioridad:** Alta | **Points:** 8

**Como** administrador, **quiero** crear y editar productos con precio, categoria, marca, imagen y codigo de barras **para** mantener actualizado el catalogo comercial.

**Criterios de aceptacion:**
- ADMIN/MANAGER pueden crear, editar, listar y desactivar productos.
- El barcode es unico cuando se informa.
- El precio de venta no puede ser negativo.
- El producto guarda online_status para controlar visibilidad publica.

**Subtareas sugeridas:**
- Crear Product entity, repository, service y admin controller.
- Implementar endpoints GET/POST/PUT/DELETE /api/admin/products.
- Crear ProductRequest, ProductSummaryDto y ProductDetailDto.
- Validar barcode unico, sale_price >= 0 y categoryId valido.
- Crear pantalla Angular admin de productos con tabla paginada.
- Crear formulario de alta/edicion con validaciones frontend.
- Agregar tests de producto duplicado, precio invalido y edicion.

---

## S1-US09 -- Publicar y pausar productos en tienda

**Epica:** EP-02 | **Prioridad:** Media | **Points:** 5

**Como** administrador, **quiero** cambiar el estado online de un producto **para** controlar que productos aparecen en la tienda sin borrarlos del sistema.

**Criterios de aceptacion:**
- El producto puede pasar por DRAFT, PUBLISHED, PAUSED y HIDDEN.
- La tienda solo muestra PUBLISHED.
- El endpoint PATCH /api/admin/products/{id}/status actualiza el estado.
- La UI muestra claramente el estado actual.

**Subtareas sugeridas:**
- Crear OnlineStatus enum y transicion controlada en ProductService.
- Implementar PATCH /api/admin/products/{id}/status.
- Filtrar GET /api/store/products por PUBLISHED.
- Agregar selector de estado en el ABM de productos.
- Agregar tests para evitar estados invalidos y verificar filtro publico.

---

## S1-US10 -- Mostrar catalogo publico inicial

**Epica:** EP-03 | **Prioridad:** Alta | **Points:** 8

**Como** cliente, **quiero** ver productos publicados con busqueda y filtro por categoria **para** conocer que vende la dietetica antes de comprar.

**Criterios de aceptacion:**
- GET /api/store/products soporta q, categoryId, branchId, page y size.
- GET /api/store/products/{id} devuelve detalle de producto.
- Solo se muestran productos PUBLISHED y activos.
- La pantalla publica permite buscar y filtrar sin login.

**Subtareas sugeridas:**
- Implementar StoreProductController con listado y detalle.
- Crear queries paginadas por nombre, categoria y estado publicado.
- Crear componentes Angular: StoreLayout, ProductGrid, ProductCard, ProductDetail.
- Implementar StoreProductService en Angular.
- Agregar estados visuales de loading, empty y error.
- Agregar tests de endpoint publico y filtro por published.

---

## S1-US11 -- Estandarizar errores y validaciones de API

**Epica:** EP-00 | **Prioridad:** Alta | **Points:** 5

**Como** desarrollador, **quiero** tener respuestas de error uniformes **para** simplificar el manejo global de errores en frontend y testing.

**Criterios de aceptacion:**
- Todas las excepciones de dominio devuelven ApiError con status, code, message, details, timestamp y path.
- Los errores de validacion devuelven VALIDATION_ERROR con campos invalidos.
- El frontend tiene interceptor global para mostrar mensajes amigables.
- No se filtran stacktraces al cliente.

**Subtareas sugeridas:**
- Crear ApiError record/clase.
- Crear DomainException base con code y status.
- Implementar @ControllerAdvice global.
- Mapear MethodArgumentNotValidException a VALIDATION_ERROR.
- Crear HttpErrorInterceptor en Angular.
- Agregar tests de formato de error.

---

## S1-US12 -- Dejar base de testing y datos demo

**Epica:** EP-00 | **Prioridad:** Media | **Points:** 5

**Como** desarrollador, **quiero** contar con pruebas y datos minimos desde el inicio **para** reducir regresiones en los flujos criticos posteriores.

**Criterios de aceptacion:**
- Existe configuracion de tests unitarios e integracion con PostgreSQL de test.
- Existe seed demo con sucursal, usuarios, categorias y productos.
- Los comandos de test estan documentados.
- La demo inicial permite login y navegacion basica del catalogo.

**Subtareas sugeridas:**
- Configurar JUnit, Mockito y Testcontainers PostgreSQL.
- Crear clase base para integration tests.
- Agregar script de seed demo con 15-20 productos representativos.
- Crear usuario admin, empleado y customer demo.
- Agregar npm scripts de lint/test/build en frontend.
- Documentar credenciales demo no productivas.

---

# Sprint 2 -- Stock, ordenes online, carrito y proveedores

**Objetivo:** Incorporar inventario real por lotes, FEFO, ordenes online pendientes de pago, carrito local, proveedores y consulta de pedidos del cliente.
**Stories:** 12 | **Points:** 75

---

## S2-US01 -- Crear modelo de stock por lotes

**Epica:** EP-05 | **Prioridad:** Alta | **Points:** 8

**Como** administrador, **quiero** registrar stock por lote, sucursal y vencimiento **para** tener trazabilidad real y soporte FEFO.

**Criterios de aceptacion:**
- Existen tablas stock_lots y stock_movements con migraciones Flyway.
- El stock disponible se calcula como SUM(quantity_available) por producto y sucursal.
- quantity_available soporta decimales numeric(12,3).
- No existe branch_product_stock ni stock_reservations.

**Subtareas sugeridas:**
- Crear V4__inventory.sql con stock_lots y stock_movements.
- Crear StockLot, StockMovement y enums de movimientos.
- Crear indices por product_id, branch_id y expiration_date.
- Implementar StockLotRepository con consultas de disponibilidad.
- Implementar endpoint GET /api/admin/stock/lots.
- Agregar tests de calculo de stock disponible.

---

## S2-US02 -- Registrar ingresos de stock

**Epica:** EP-05 | **Prioridad:** Alta | **Points:** 5

**Como** administrador, **quiero** cargar nuevos lotes de productos por sucursal **para** actualizar disponibilidad real del negocio.

**Criterios de aceptacion:**
- POST /api/admin/stock/lots crea lote con producto, sucursal, cantidad, lote, vencimiento y costo opcional.
- Cada ingreso genera stock_movement PURCHASE_ENTRY.
- No se aceptan cantidades negativas o cero.
- La UI permite cargar vencimiento opcional.

**Subtareas sugeridas:**
- Crear CreateStockLotRequest y StockLotDto.
- Implementar InventoryService.createStockLot().
- Registrar movimiento PURCHASE_ENTRY asociado al lote.
- Crear pantalla Angular de ingreso de stock.
- Mostrar stock total actual luego de cargar el lote.
- Agregar tests de ingreso valido e invalido.

---

## S2-US03 -- Implementar politica FEFO testeable

**Epica:** EP-05 | **Prioridad:** Alta | **Points:** 8

**Como** desarrollador, **quiero** descontar stock priorizando lotes que vencen primero **para** cumplir la regla de negocio central de inventario.

**Criterios de aceptacion:**
- La politica ordena por expiration_date ascendente con NULLS LAST.
- Puede descontar de multiples lotes para cubrir una cantidad.
- Falla con INSUFFICIENT_STOCK si no alcanza el total.
- La logica pura puede probarse sin levantar Spring.

**Subtareas sugeridas:**
- Crear FefoStockDeductionPolicy.
- Definir estructura DeductionPlan con lotId y quantityToDeduct.
- Implementar calculo sobre lista de lotes disponibles.
- Cubrir casos: un lote, varios lotes, lote sin vencimiento, stock insuficiente y cantidad decimal.
- Documentar criterio FEFO dentro del modulo inventory.

---

## S2-US04 -- Registrar ajustes manuales y consumos internos

**Epica:** EP-05 | **Prioridad:** Alta | **Points:** 5

**Como** administrador, **quiero** ajustar stock con motivo obligatorio **para** corregir diferencias manteniendo trazabilidad.

**Criterios de aceptacion:**
- POST /api/admin/stock/adjustments permite ajuste positivo o negativo con reason obligatorio.
- No se permite dejar un lote con stock negativo.
- Los movimientos quedan registrados como MANUAL_ADJUSTMENT o INTERNAL_CONSUMPTION.
- EMPLOYEE puede consultar stock pero no realizar ajustes.

**Subtareas sugeridas:**
- Crear StockAdjustmentRequest.
- Implementar validacion de motivo obligatorio.
- Implementar ajuste sobre lote especifico o producto/sucursal con FEFO para negativos.
- Registrar StockMovement con created_by_user_id.
- Crear listado de movimientos filtrable.
- Agregar tests de ajuste negativo sin stock y reason vacio.

---

## S2-US05 -- Mostrar disponibilidad real en tienda

**Epica:** EP-03 | **Prioridad:** Alta | **Points:** 5

**Como** cliente, **quiero** ver stock disponible por sucursal en catalogo y detalle **para** evitar agregar productos agotados al carrito.

**Criterios de aceptacion:**
- La tienda consulta disponibilidad desde stock_lots por branchId.
- El detalle de producto muestra disponible/agotado segun stock.
- Si no hay stock, el boton de agregar al carrito queda deshabilitado.
- El backend no expone costos ni datos internos.

**Subtareas sugeridas:**
- Extender ProductSummaryDto y ProductDetailDto con availableStock.
- Optimizar consulta de disponibilidad para listado paginado.
- Actualizar ProductCard y ProductDetail con stock visible.
- Validar branchId requerido o usar sucursal default del MVP.
- Agregar tests de producto publicado sin stock.

---

## S2-US06 -- Crear modelo de ordenes unificadas

**Epica:** EP-06 | **Prioridad:** Alta | **Points:** 8

**Como** desarrollador, **quiero** representar ventas POS y pedidos online en una misma entidad **para** evitar duplicacion de logica comercial.

**Criterios de aceptacion:**
- Existen tablas orders y order_items con type POS/ONLINE.
- OrderItem guarda snapshot de nombre, barcode, precio y costo cuando exista.
- ONLINE requiere customer_user_id y POS requiere created_by_user_id.
- fulfillment_type queda en PICKUP para el MVP.

**Subtareas sugeridas:**
- Crear V5__orders.sql con orders y order_items.
- Crear entidades Order y OrderItem.
- Crear enums OrderType, OrderStatus y FulfillmentType.
- Implementar OrderNumberGenerator.
- Crear repositorios y DTOs OrderSummaryDto/OrderDetailDto.
- Agregar constraints y tests de reglas ONLINE/POS.

---

## S2-US07 -- Crear base de pagos unificados

**Epica:** EP-09 | **Prioridad:** Alta | **Points:** 5

**Como** desarrollador, **quiero** registrar pagos online y presenciales en una tabla comun **para** tener trazabilidad y reportes consistentes.

**Criterios de aceptacion:**
- Existe tabla payments con provider, method, status, amount y referencias externas.
- El payment online puede tener cash_session_id null.
- El payment presencial queda preparado para asociarse a cash_session_id en Sprint 3.
- No se almacena informacion sensible de tarjetas.

**Subtareas sugeridas:**
- Crear V6__payments.sql.
- Crear Payment entity y enums PaymentProvider, PaymentMethod, PaymentStatus.
- Crear PaymentRepository y PaymentService base.
- Agregar relacion Order -> Payments.
- Implementar consulta de payments por order.
- Agregar tests de constraints de metodo, proveedor y monto.

---

## S2-US08 -- Crear orden online pendiente de pago

**Epica:** EP-06 | **Prioridad:** Alta | **Points:** 8

**Como** cliente, **quiero** confirmar mi carrito y crear una orden online **para** iniciar el checkout sin descontar stock todavia.

**Criterios de aceptacion:**
- POST /api/customer/orders requiere rol CUSTOMER.
- Valida que haya stock suficiente al momento de crear la orden.
- Crea order ONLINE con status PENDING_PAYMENT.
- Crea payment MERCADO_PAGO/CHECKOUT_PRO con status PENDING.
- No descuenta stock en esta etapa.

**Subtareas sugeridas:**
- Crear CustomerOrderController.
- Implementar CreateOnlineOrderRequest con items productId/quantity.
- Validar productos activos y publicados.
- Validar stock disponible por sucursal sin modificar lotes.
- Crear snapshots de cliente y producto en la orden.
- Calcular subtotal, descuentos en 0 y total.
- Agregar tests de stock insuficiente, producto inexistente y orden exitosa.

---

## S2-US09 -- Implementar carrito local en Angular

**Epica:** EP-03 | **Prioridad:** Alta | **Points:** 5

**Como** cliente, **quiero** agregar productos al carrito local **para** preparar una compra sin persistir carrito en base.

**Criterios de aceptacion:**
- El carrito se guarda en localStorage.
- Permite agregar, quitar, modificar cantidades y vaciar.
- Revalida stock antes de confirmar compra.
- El carrito no queda asociado a usuario en base de datos.

**Subtareas sugeridas:**
- Crear CartService con Angular Signals.
- Crear modelo CartItem con productId, name, price, imageUrl y quantity.
- Crear componentes CartDrawer/CartPage.
- Persistir y recuperar carrito desde localStorage.
- Validar cantidad mayor a cero y no superior al disponible conocido.
- Agregar tests unitarios del servicio de carrito.

---

## S2-US10 -- Completar flujo frontend de confirmacion online

**Epica:** EP-03 | **Prioridad:** Alta | **Points:** 5

**Como** cliente, **quiero** confirmar mi carrito autenticado **para** generar una orden pendiente de pago.

**Criterios de aceptacion:**
- Si el usuario no esta autenticado, se lo redirige a login antes de confirmar.
- El checkout crea una orden ONLINE PENDING_PAYMENT desde el carrito local.
- Al crear la orden se muestra numero de orden y estado.
- El carrito no se vacia hasta que el pago sea iniciado/confirmado.

**Subtareas sugeridas:**
- Crear CheckoutPage protegida por AuthGuard CUSTOMER.
- Integrar CartService con CustomerOrderService.
- Mostrar resumen de compra, cantidades y total.
- Manejar errores INSUFFICIENT_STOCK y PRODUCT_NOT_FOUND.
- Crear pantalla de orden creada pendiente de pago.
- Agregar tests de componente/servicio para confirmacion.

---

## S2-US11 -- Gestionar proveedores y costos manuales

**Epica:** EP-10 | **Prioridad:** Media | **Points:** 8

**Como** administrador, **quiero** registrar proveedores y costos por producto **para** tener referencia de reposicion y margen sin importar listas externas.

**Criterios de aceptacion:**
- ADMIN puede crear y editar proveedores.
- ADMIN puede asociar producto-proveedor con costo actual y SKU proveedor.
- El costo no modifica automaticamente el precio de venta.
- La importacion automatica queda fuera del MVP.

**Subtareas sugeridas:**
- Crear V3__suppliers.sql con suppliers y supplier_products.
- Crear entidades Supplier y SupplierProduct.
- Implementar endpoints /api/admin/suppliers.
- Implementar endpoints /api/admin/supplier-products.
- Crear pantallas Angular de proveedores y asociacion con producto.
- Agregar tests de CUIT duplicado, costo negativo y asociacion.

---

## S2-US12 -- Consultar pedidos propios del cliente

**Epica:** EP-03 | **Prioridad:** Media | **Points:** 5

**Como** cliente, **quiero** ver mis pedidos y su estado **para** saber cuando puedo retirar la compra.

**Criterios de aceptacion:**
- GET /api/customer/orders lista solo ordenes del CUSTOMER autenticado.
- GET /api/customer/orders/{id} impide ver pedidos ajenos.
- El detalle incluye items, total, estado de orden y estado del payment.
- La UI muestra historial de pedidos del cliente.

**Subtareas sugeridas:**
- Implementar endpoints de listado y detalle customer.
- Agregar validacion de ownership por customer_user_id.
- Crear CustomerOrdersPage y CustomerOrderDetailPage.
- Mostrar estados PENDING_PAYMENT, PAID, PREPARING, READY, DELIVERED, CANCELLED, PAYMENT_FAILED y STOCK_CONFLICT.
- Agregar tests de acceso prohibido a pedido ajeno.

---

# Sprint 3 -- Mercado Pago, caja operativa y POS

**Objetivo:** Completar los flujos transaccionales principales: Mercado Pago, webhook, descuento FEFO online, caja operativa y venta presencial POS.
**Stories:** 12 | **Points:** 75

---

## S3-US01 -- Crear adaptador de Mercado Pago

**Epica:** EP-04 | **Prioridad:** Alta | **Points:** 5

**Como** desarrollador, **quiero** encapsular la integracion con Mercado Pago detras de una interfaz **para** poder testear con mock y cambiar implementacion sin redisenar pagos.

**Criterios de aceptacion:**
- Existe PaymentGateway con createPreference, checkPayment y verifyWebhookSignature.
- Existe MercadoPagoGateway como implementacion real/configurable.
- Los tests pueden usar FakePaymentGateway.
- Las credenciales se leen desde variables de entorno.

**Subtareas sugeridas:**
- Crear interfaz PaymentGateway.
- Crear DTO interno PaymentPreferenceResult con initPoint y preferenceId.
- Crear MercadoPagoGateway con configuracion externa.
- Crear FakePaymentGateway para tests/dev.
- Agregar properties MP_ACCESS_TOKEN, MP_WEBHOOK_SECRET, MP_SUCCESS_URL, MP_FAILURE_URL.
- Documentar como usar sandbox/mock.

---

## S3-US02 -- Crear preferencia de pago Checkout Pro

**Epica:** EP-04 | **Prioridad:** Alta | **Points:** 5

**Como** cliente, **quiero** iniciar el pago de mi orden en Mercado Pago **para** completar la compra en una pasarela segura.

**Criterios de aceptacion:**
- POST /api/customer/orders/{orderId}/checkout/mp crea o reutiliza preferencia.
- El endpoint es idempotente si la orden ya tiene provider_preference_id.
- Solo se permite para ordenes propias en PENDING_PAYMENT.
- Devuelve initPoint y preferenceId.

**Subtareas sugeridas:**
- Crear MercadoPagoService.createCheckoutPreference().
- Validar estado de orden PENDING_PAYMENT.
- Validar ownership CUSTOMER.
- Guardar provider_preference_id y external_reference en payment.
- Mapear datos de order/items al request de MP.
- Agregar tests de preferencia nueva, preferencia existente y orden invalida.

---

## S3-US03 -- Procesar webhook de Mercado Pago con idempotencia

**Epica:** EP-04 | **Prioridad:** Alta | **Points:** 8

**Como** sistema, **quiero** recibir confirmaciones de pago desde Mercado Pago **para** actualizar orden y pago sin duplicar operaciones.

**Criterios de aceptacion:**
- POST /api/webhooks/mercadopago verifica firma antes de procesar.
- Consulta el estado real del pago en el proveedor/gateway.
- Si el mismo provider_payment_id ya fue procesado, responde OK sin duplicar stock_movements.
- REJECTED actualiza payment y order a PAYMENT_FAILED.

**Subtareas sugeridas:**
- Crear endpoint publico MercadoPagoWebhookController.
- Implementar verificacion de firma en gateway.
- Implementar busqueda por provider_payment_id/provider_preference_id/external_reference.
- Agregar control idempotente antes de modificar orden o stock.
- Mapear estados MP a PaymentStatus interno.
- Registrar metadata JSONB del webhook sin datos sensibles.
- Agregar integration tests de webhook duplicado y rechazado.

---

## S3-US04 -- Descontar stock FEFO al aprobar pago online

**Epica:** EP-05 | **Prioridad:** Alta | **Points:** 8

**Como** sistema, **quiero** descontar stock cuando Mercado Pago aprueba la orden **para** mantener sincronizado stock real y tienda online.

**Criterios de aceptacion:**
- Al aprobar pago online, se bloquean lotes con SELECT FOR UPDATE.
- Se descuenta stock FEFO por cada item.
- Se crean movimientos ONLINE_ORDER con order_id.
- La order pasa a PAID y payment a APPROVED.
- Si no hay stock suficiente, la order pasa a STOCK_CONFLICT y no queda stock negativo.

**Subtareas sugeridas:**
- Implementar InventoryService.deductForOnlineOrder() transaccional.
- Crear consulta repository con bloqueo pesimista ordenada por vencimiento.
- Registrar StockMovement por lote afectado.
- Actualizar paid_at y approved_at.
- Implementar manejo STOCK_CONFLICT.
- Agregar tests concurrentes basicos de no sobreventa.
- Agregar tests de descuento de varios lotes.

---

## S3-US05 -- Integrar checkout frontend con Mercado Pago

**Epica:** EP-03 | **Prioridad:** Alta | **Points:** 5

**Como** cliente, **quiero** ser redirigido a Mercado Pago y volver a ver el resultado **para** completar la compra online de punta a punta.

**Criterios de aceptacion:**
- La UI llama al endpoint de checkout y redirige a initPoint.
- Existe pantalla de resultado de pago que consulta el estado real de la orden.
- El cliente ve mensajes claros para pago pendiente, aprobado, rechazado o conflicto de stock.
- No se muestra informacion interna de pagos.

**Subtareas sugeridas:**
- Crear CustomerCheckoutService.
- Agregar boton Pagar con Mercado Pago en orden pendiente.
- Implementar window.location.href hacia initPoint.
- Crear PaymentResultPage para success/failure/pending.
- Consultar GET /api/customer/orders/{id} al volver.
- Manejar estado asincronico donde el webhook aun no impacto.

---

## S3-US06 -- Abrir caja operativa por sucursal

**Epica:** EP-07 | **Prioridad:** Alta | **Points:** 8

**Como** empleado, **quiero** abrir caja con monto inicial de efectivo **para** comenzar ventas presenciales controlando efectivo.

**Criterios de aceptacion:**
- ADMIN/MANAGER/EMPLOYEE pueden abrir caja.
- Solo puede existir una caja OPEN por sucursal.
- La caja guarda opened_by_user_id, branch_id, opening_cash_amount y opening_notes.
- GET /api/admin/cash-sessions/current devuelve la caja abierta actual.

**Subtareas sugeridas:**
- Crear V7__cash.sql con cash_sessions y cash_movements.
- Crear CashSession entity, repository y status enum.
- Implementar CashService.openCashSession().
- Validar una caja abierta por sucursal.
- Crear endpoints open/current.
- Crear UI de apertura de caja.
- Agregar tests de caja duplicada y apertura por empleado.

---

## S3-US07 -- Registrar movimientos manuales de caja

**Epica:** EP-07 | **Prioridad:** Media | **Points:** 5

**Como** empleado, **quiero** registrar ingresos, egresos o ajustes manuales **para** mantener trazabilidad del efectivo y movimientos no asociados a ventas.

**Criterios de aceptacion:**
- Solo se pueden registrar movimientos si la caja esta OPEN.
- Todo movimiento exige motivo.
- Se registran type CASH_IN/CASH_OUT/ADJUSTMENT y method CASH/TRANSFER/OTHER.
- El movimiento queda asociado al usuario que lo creo.

**Subtareas sugeridas:**
- Crear CashMovement entity y enums.
- Implementar POST /api/admin/cash-sessions/{id}/movements.
- Validar reason obligatorio y amount distinto de cero.
- Actualizar detalle de caja con movimientos manuales.
- Crear formulario Angular de movimiento manual.
- Agregar tests de caja cerrada, reason vacio y usuario creador.

---

## S3-US08 -- Cerrar caja con arqueo de efectivo

**Epica:** EP-07 | **Prioridad:** Alta | **Points:** 5

**Como** empleado, **quiero** cerrar caja informando efectivo contado **para** comparar efectivo esperado contra fisico y justificar diferencias.

**Criterios de aceptacion:**
- El cierre calcula expected_cash_amount usando apertura + pagos CASH + movimientos CASH.
- El cierre muestra totales informativos por CASH, QR, TRANSFER, DEBIT_CARD y CREDIT_CARD.
- Si countedCashAmount difiere de expected, exige cashDifferenceReason.
- La diferencia no bloquea el cierre si esta justificada.

**Subtareas sugeridas:**
- Crear CashCloseCalculator.
- Implementar POST /api/admin/cash-sessions/{id}/close.
- Calcular totalsByMethod desde payments asociados a la caja.
- Persistir counted, expected, difference, reason, closed_by_user_id y closed_at.
- Crear UI de cierre con resumen de metodos y campo de efectivo contado.
- Agregar tests de cierre sin diferencia, con diferencia sin razon y con diferencia justificada.

---

## S3-US09 -- Buscar productos en POS por nombre o codigo de barras

**Epica:** EP-08 | **Prioridad:** Media | **Points:** 5

**Como** empleado, **quiero** buscar productos rapidamente en la venta presencial **para** reducir tiempo operativo y soportar lector como input teclado.

**Criterios de aceptacion:**
- El POS permite buscar por nombre o barcode.
- La busqueda muestra precio y stock disponible en la sucursal del empleado.
- El lector de codigo de barras funciona como entrada de teclado.
- No se pueden agregar productos sin stock.

**Subtareas sugeridas:**
- Crear endpoint especifico o reutilizar catalogo admin con filtro q/barcode.
- Optimizar indice de barcode.
- Crear POSProductSearchComponent.
- Agregar input enfocado para scanner.
- Mostrar stock disponible y precio.
- Agregar tests de busqueda por barcode.

---

## S3-US10 -- Crear venta presencial transaccional

**Epica:** EP-08 | **Prioridad:** Alta | **Points:** 8

**Como** empleado, **quiero** cobrar una venta presencial con distintos medios de pago **para** registrar venta, pago, caja y descuento de stock en una operacion segura.

**Criterios de aceptacion:**
- POST /api/admin/pos/sales exige caja abierta.
- Acepta metodos CASH, QR, TRANSFER, DEBIT_CARD y CREDIT_CARD.
- Crea order POS, order_items, payment APPROVED con cash_session_id y stock_movements POS_SALE.
- Descuenta stock FEFO dentro de la misma transaccion.
- Si no hay stock o caja abierta, no se crea venta parcial.

**Subtareas sugeridas:**
- Implementar PosSaleService.createSale() con @Transactional.
- Validar caja OPEN de la sucursal del usuario.
- Bloquear lotes con FOR UPDATE y aplicar FEFO.
- Crear order POS con status COMPLETED.
- Crear payment provider MANUAL segun metodo.
- Crear snapshots de items.
- Registrar movimientos POS_SALE por lote.
- Agregar integration tests de venta cash, QR, stock insuficiente y sin caja.

---

## S3-US11 -- Construir pantalla POS completa

**Epica:** EP-08 | **Prioridad:** Alta | **Points:** 8

**Como** empleado, **quiero** armar una venta, modificar cantidades y cobrar **para** operar ventas presenciales desde una interfaz rapida.

**Criterios de aceptacion:**
- La pantalla POS permite agregar productos, modificar cantidades, quitar items y ver total.
- Antes de cobrar verifica caja abierta actual.
- Permite seleccionar metodo de pago.
- Muestra comprobante/resumen simple al finalizar.

**Subtareas sugeridas:**
- Crear AdminPosPage.
- Integrar POSProductSearchComponent con carrito POS en memoria.
- Implementar selector de metodo de pago.
- Mostrar total y validaciones de stock/cantidad.
- Consumir POST /api/admin/pos/sales.
- Mostrar resultado con numero de orden y metodo de pago.
- Agregar manejo de errores CASH_SESSION_REQUIRED e INSUFFICIENT_STOCK.

---

## S3-US12 -- Probar flujos criticos de pagos, caja y POS

**Epica:** EP-00 | **Prioridad:** Alta | **Points:** 5

**Como** desarrollador, **quiero** tener cobertura de los flujos transaccionales principales **para** evitar regresiones en las partes mas riesgosas del MVP.

**Criterios de aceptacion:**
- Hay integration tests para webhook aprobado, webhook duplicado y stock conflict.
- Hay integration tests para abrir/cerrar caja y venta POS.
- Hay unit tests para CashCloseCalculator y FefoStockDeductionPolicy.
- Las pruebas corren en el pipeline local documentado.

**Subtareas sugeridas:**
- Crear fixtures de productos, lotes, usuarios y caja.
- Testear webhook aprobado con descuento FEFO.
- Testear webhook duplicado sin doble descuento.
- Testear POS con caja abierta y sin caja.
- Testear cierre de caja con diferencia y razon obligatoria.
- Agregar comando unico backend test.

---

# Sprint 4 -- Pedidos, reportes, seguridad y despliegue

**Objetivo:** Cerrar operacion backoffice y calidad final: estados de pedidos, cancelaciones, reportes, recomendaciones, auditoria, seguridad, E2E, demo y documentacion.
**Stories:** 12 | **Points:** 75

---

## S4-US01 -- Gestionar preparacion y retiro de pedidos online

**Epica:** EP-06 | **Prioridad:** Alta | **Points:** 8

**Como** empleado, **quiero** cambiar estados de pedidos online hasta entregarlos **para** organizar el retiro en sucursal con trazabilidad.

**Criterios de aceptacion:**
- El empleado puede pasar PAID -> PREPARING -> READY -> DELIVERED.
- No se puede preparar una orden sin pago aprobado.
- DELIVERED no descuenta stock nuevamente.
- Se registran timestamps prepared_at y delivered_at.

**Subtareas sugeridas:**
- Implementar PATCH /api/admin/orders/{id}/prepare.
- Implementar PATCH /api/admin/orders/{id}/ready.
- Implementar PATCH /api/admin/orders/{id}/delivered.
- Validar transiciones con OrderStatePolicy.
- Crear UI admin de cambio de estado.
- Agregar tests de transiciones validas e invalidas.

---

## S4-US02 -- Cancelar orden y revertir stock

**Epica:** EP-06 | **Prioridad:** Alta | **Points:** 8

**Como** administrador, **quiero** cancelar una orden con motivo **para** corregir pedidos problematicos sin perder trazabilidad.

**Criterios de aceptacion:**
- PATCH /api/admin/orders/{id}/cancel exige reason.
- Si la orden ya descont stock, se revierte contra los lotes originales.
- Se crean movimientos CANCELLATION_RETURN.
- No se puede cancelar una orden DELIVERED.

**Subtareas sugeridas:**
- Implementar OrderCancellationService transaccional.
- Buscar stock_movements originales por order_id.
- Restaurar quantity_available en los mismos stock_lot_id.
- Crear movimientos inversos CANCELLATION_RETURN.
- Actualizar status CANCELLED y cancellation_reason.
- Actualizar payment CANCELLED o REFUNDED segun escenario.
- Agregar tests de cancelacion PAID, PREPARING, READY y estado invalido.

---

## S4-US03 -- Administrar pedidos desde backoffice

**Epica:** EP-06 | **Prioridad:** Alta | **Points:** 5

**Como** empleado, **quiero** listar, filtrar y ver detalle de pedidos online y ventas POS **para** operar preparacion, entrega y control de ordenes.

**Criterios de aceptacion:**
- GET /api/admin/orders filtra por status, branchId, type y rango de fechas.
- El detalle muestra items, pagos y datos snapshot del cliente.
- MANAGER/EMPLOYEE ven ordenes de su sucursal; ADMIN puede ver global.
- La UI permite acceder a acciones permitidas segun estado.

**Subtareas sugeridas:**
- Implementar filtros de OrderRepository.
- Crear AdminOrdersPage con tabla paginada.
- Crear AdminOrderDetailPage.
- Mostrar pagos asociados y estado actual.
- Mostrar acciones prepare/ready/delivered/cancel segun estado.
- Agregar tests de filtros y permisos por rol/sucursal.

---

## S4-US04 -- Crear dashboard operativo del dia

**Epica:** EP-11 | **Prioridad:** Media | **Points:** 8

**Como** administrador, **quiero** ver metricas principales del negocio **para** tomar decisiones rapidas con datos actuales.

**Criterios de aceptacion:**
- GET /api/admin/reports/dashboard devuelve ventas del dia, online, POS, pendingOrders, lowStockProducts, expiringLots y topProducts.
- ADMIN ve datos globales; MANAGER ve datos de su sucursal.
- El dashboard carga rapido y no bloquea tareas criticas.
- La UI muestra tarjetas claras y tabla de productos top.

**Subtareas sugeridas:**
- Crear ReportService.dashboard().
- Crear queries agregadas para ventas por tipo y dia.
- Crear query de productos mas vendidos.
- Crear query de pedidos pendientes.
- Crear DashboardPage Angular con cards y tabla.
- Agregar tests de agregacion por sucursal y rol.

---

## S4-US05 -- Generar reporte de cierre de caja

**Epica:** EP-11 | **Prioridad:** Media | **Points:** 5

**Como** administrador, **quiero** ver detalle de un cierre con totales por medio de pago **para** auditar caja y entender diferencias de efectivo.

**Criterios de aceptacion:**
- GET /api/admin/reports/cash-session/{id} devuelve sesion, pagos, totales por metodo, efectivo esperado, contado, diferencia y razon.
- El reporte separa arqueo de efectivo de totales informativos no cash.
- La UI permite consultar historial y detalle de cierres.

**Subtareas sugeridas:**
- Implementar CashReportDto.
- Crear query de totalsByMethod desde payments.
- Crear pantalla CashSessionHistoryPage.
- Crear pantalla CashSessionDetailReportPage.
- Mostrar apertura, cierre, operador y movimientos manuales.
- Agregar tests de totales por CASH/QR/TRANSFER/TARJETAS.

---

## S4-US06 -- Implementar recomendaciones por reglas

**Epica:** EP-11 | **Prioridad:** Media | **Points:** 8

**Como** administrador, **quiero** recibir sugerencias de reposicion, vencimientos y rotacion **para** detectar problemas operativos sin revisar todo manualmente.

**Criterios de aceptacion:**
- GET /api/admin/recommendations devuelve LOW_STOCK, EXPIRING_SOON, HIGH_ROTATION y NO_MOVEMENT cuando hay datos suficientes.
- No recomienda productos sin stock al cliente ni inventa productos.
- No expone costos ni margenes.
- Cada recomendacion incluye mensaje y urgencia.

**Subtareas sugeridas:**
- Crear RecommendationService rule-based.
- Implementar regla LOW_STOCK usando products.minimum_stock y stock disponible.
- Implementar regla EXPIRING_SOON con lotes proximos a vencer.
- Implementar regla HIGH_ROTATION con order_items recientes.
- Implementar regla NO_MOVEMENT con productos sin ventas recientes.
- Crear panel Angular de recomendaciones.
- Agregar tests de cada regla con datos controlados.

---

## S4-US07 -- Auditar acciones criticas

**Epica:** EP-00 | **Prioridad:** Alta | **Points:** 5

**Como** administrador, **quiero** tener registro de cambios sensibles **para** poder explicar diferencias, cancelaciones y cambios de datos.

**Criterios de aceptacion:**
- Se registran cambios de precio, ajustes de stock, cancelaciones, apertura/cierre de caja, movimientos de caja, confirmacion de pago y altas/bajas de usuarios internos.
- Cada audit_log guarda usuario, accion, entidad, entity_id, descripcion y fecha.
- No registra datos sensibles de tarjetas o contrasenas.
- Existe consulta administrativa basica de auditoria.

**Subtareas sugeridas:**
- Crear V9__audit.sql y AuditLog entity.
- Crear AuditService.record().
- Integrar auditoria en ProductService para cambios de precio.
- Integrar auditoria en InventoryService para ajustes.
- Integrar auditoria en CashService para apertura/cierre/movimientos.
- Integrar auditoria en Payment/Webhook para confirmaciones.
- Crear endpoint GET /api/admin/audit-logs con filtros basicos.
- Agregar tests de generacion de audit logs.

---

## S4-US08 -- Pulir seguridad, permisos y guards

**Epica:** EP-00 | **Prioridad:** Alta | **Points:** 5

**Como** desarrollador, **quiero** cerrar accesos indebidos por rol y sucursal **para** evitar exposicion de modulos internos a usuarios incorrectos.

**Criterios de aceptacion:**
- CUSTOMER no accede a /api/admin/** ni rutas backoffice.
- EMPLOYEE no accede a reportes ni ABMs restringidos.
- MANAGER opera solo su sucursal.
- Angular oculta menus segun rol ademas de validar en backend.

**Subtareas sugeridas:**
- Revisar @PreAuthorize en todos los controllers.
- Crear util de seguridad para branch scope.
- Completar AdminGuard y CustomerGuard en Angular.
- Implementar menu dinamico por rol.
- Agregar tests de acceso prohibido por rol.
- Validar que los endpoints customer no expongan pedidos ajenos.

---

## S4-US09 -- Mejorar usabilidad responsive del MVP

**Epica:** EP-00 | **Prioridad:** Media | **Points:** 5

**Como** usuario, **quiero** usar las pantallas principales en escritorio y movil **para** hacer viable el uso real en comercio y la demo ante tutor.

**Criterios de aceptacion:**
- Catalogo, carrito, login, pedidos, POS, caja y dashboard son usables en pantallas comunes.
- Las tablas grandes tienen paginacion o diseno compacto.
- Las alertas criticas son visibles en el panel principal.
- Los formularios muestran validaciones claras.

**Subtareas sugeridas:**
- Revisar layout publico de tienda en mobile.
- Revisar layout backoffice y navegacion lateral/topbar.
- Agregar estados empty/loading/error consistentes.
- Optimizar formularios de stock, caja y POS.
- Agregar confirmaciones para acciones destructivas.
- Corregir problemas de accesibilidad basicos.

---

## S4-US10 -- Cubrir flujos E2E principales

**Epica:** EP-00 | **Prioridad:** Alta | **Points:** 8

**Como** desarrollador, **quiero** validar los caminos completos del MVP **para** demostrar que el sistema funciona de punta a punta.

**Criterios de aceptacion:**
- Existe prueba E2E o guion automatizado para registro/login/customer checkout mock.
- Existe prueba E2E o guion automatizado para caja + POS + cierre.
- Existe prueba E2E o guion automatizado para preparacion y entrega de pedido.
- Los resultados quedan documentados para defensa/demo.

**Subtareas sugeridas:**
- Definir herramienta E2E o scripts de prueba manual reproducibles.
- Crear dataset estable de demo.
- Probar flujo customer: registro, catalogo, carrito, orden, checkout mock, pedido.
- Probar flujo empleado: abrir caja, vender POS, cerrar caja.
- Probar flujo admin: stock, cancelacion, reportes y recomendaciones.
- Documentar bugs encontrados y correcciones aplicadas.

---

## S4-US11 -- Preparar despliegue de demo

**Epica:** EP-00 | **Prioridad:** Alta | **Points:** 5

**Como** desarrollador, **quiero** desplegar una version demostrable del MVP **para** mostrar el sistema funcionando fuera del entorno local.

**Criterios de aceptacion:**
- Existe configuracion de demo con Docker Compose.
- Las variables sensibles se gestionan por entorno.
- Uploads de imagenes quedan servidos como estaticos.
- La demo permite ejecutar los flujos principales con usuarios seed.

**Subtareas sugeridas:**
- Crear Dockerfile backend.
- Crear build de frontend y configuracion de servidor estatico/proxy.
- Configurar CORS para ambiente demo.
- Configurar variables de Mercado Pago sandbox/mock.
- Crear script de migracion/seed para demo.
- Documentar pasos de despliegue y rollback simple.
- Validar demo desde navegador limpio.

---

## S4-US12 -- Cerrar documentacion academica y evidencia Jira

**Epica:** EP-00 | **Prioridad:** Media | **Points:** 5

**Como** desarrollador, **quiero** dejar documentacion final alineada al MVP implementado **para** facilitar revision del tutor y defensa del proyecto.

**Criterios de aceptacion:**
- La documentacion refleja el alcance real implementado.
- Quedan capturas o evidencias de los flujos principales.
- El README explica arquitectura, ejecucion, usuarios demo y decisiones.
- El tablero Jira tiene epicas, historias y tareas actualizadas con estados reales.

**Subtareas sugeridas:**
- Actualizar README funcional y tecnico.
- Actualizar documentacion de endpoints implementados.
- Agregar diagrama simple de arquitectura final.
- Preparar guion de demo: customer, empleado, admin.
- Adjuntar capturas de catalogo, checkout, caja, POS, reportes y recomendaciones.
- Revisar que Jira tenga dependencias, story points y criterios de aceptacion completos.

---

> [!seealso] Notas relacionadas
> - [[Epicas]]
> - [[MVP]]
> - [[Roadmap]]
> - Volver a [[_Index]]
