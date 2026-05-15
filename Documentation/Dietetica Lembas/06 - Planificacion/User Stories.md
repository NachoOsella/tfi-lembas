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

**Backend:**
- Crear estructura de proyecto backend con modulos: auth, users, catalog, inventory, orders, payments, cash, suppliers, reports, audit y shared.
- Crear paquete shared para DTOs comunes, excepciones y utilidades.
- Configurar perfiles dev/test en application.yml.
- Configurar dependencias base (Spring Boot, JPA, Security, Flyway, Validation).
- Definir convencion de nombres para endpoints, DTOs y servicios.

**Frontend:**
- Inicializar proyecto Angular 18+ con standalone components y routing.
- Crear estructura de carpetas por features: store, customer, admin, auth y shared.
- Crear modulo shared con componentes base: LoadingSpinner, EmptyState, ErrorAlert, ConfirmDialog.
- Crear layout base para tienda publica (StoreLayout: header con logo + nav + icono carrito, footer).
- Crear layout base para backoffice (AdminLayout: sidebar colapsable, topbar con usuario, breadcrumbs).
- Configurar Angular Material y tema base (colores, tipografia, espaciados).
- Configurar archivos environment.ts y environment.development.ts con apiUrl.
- Configurar proxy para desarrollo local (proxy.conf.json hacia backend en :8080).
- Configurar ESLint, Prettier y convencion de nombres para componentes, servicios y rutas.
- Agregar README con comandos de ejecucion, estructura y convenciones de ramas.

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

**Backend / Infra:**
- Crear docker-compose.yml con PostgreSQL 16 y red interna.
- Definir variables DB_HOST, DB_PORT, DB_NAME, DB_USER, DB_PASSWORD y JWT_SECRET.
- Configurar application-dev.yml y application-test.yml.
- Agregar script de reset de base local para desarrollo.
- Documentar comandos: up, down, logs, migrate y seed.

**Frontend:**
- Agregar script npm run docker:up que levante solo la BD para desarrollo frontend.
- Verificar que el frontend se conecte al backend en Docker sin CORS issues.

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

**Backend / DB:**
- Configurar dependencia Flyway en Spring Boot.
- Crear V1__core.sql con branches y users.
- Crear V2__catalog.sql con categories y products.
- Agregar CHECK para role ADMIN/MANAGER/EMPLOYEE/CUSTOMER.
- Agregar CHECK para online_status DRAFT/PUBLISHED/PAUSED/HIDDEN.
- Crear V10__seed_data.sql parcial con sucursal Centro, admin demo y categorias base.

**Frontend:**
- No requiere subtareas frontend directas (es puramente DB).

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

**Backend:**
- Crear User entity, UserRepository y Role enum.
- Crear RegisterRequest, AuthResponse y UserDto.
- Implementar AuthService.registerCustomer().
- Validar email, password, firstName, lastName y phone opcional.
- Crear tests unitarios de registro exitoso, email duplicado y password hash.

**Frontend:**
- Crear RegisterPageComponent con formulario de registro (nombre, apellido, email, password, confirmar password, telefono opcional).
- Agregar validaciones en tiempo real: email formato, password >= 8 caracteres, confirmacion coincide.
- Mostrar errores especificos del backend: EMAIL_DUPLICATED, VALIDATION_ERROR.
- Crear AuthService con metodo register().
- Redirigir a login con mensaje de exito al registrarse.
- Agregar ruta /auth/register con layout publico.
- Manejar estados: loading (boton deshabilitado + spinner), error (mensaje claro), exito (redirect).
- Agregar tests unitarios del componente (formulario valido, invalido, errores backend).

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

**Backend:**
- Configurar Spring Security stateless y BCryptPasswordEncoder.
- Implementar JwtService para generar y validar token de 24h.
- Implementar JwtAuthenticationFilter.
- Configurar rutas publicas /api/auth/**, /api/store/**, /api/webhooks/** y /uploads/**.
- Crear AuthController con login y me.
- Agregar tests de login, token invalido y usuario deshabilitado.

**Frontend:**
- Crear LoginPageComponent con formulario de email + password.
- Crear AuthService con metodos login(), logout(), getToken(), isAuthenticated(), getUserRole().
- Almacenar token en localStorage y decodificar payload para datos basicos del usuario.
- Crear AuthInterceptor que adjunte token JWT en cada request.
- Crear AuthGuard con logica de redireccion segun autenticacion y rol.
- Crear AdminGuard (redirige a /store si no es ADMIN/MANAGER/EMPLOYEE).
- Crear CustomerGuard (redirige a /auth/login si no es CUSTOMER).
- Redirigir a /store si el usuario ya esta autenticado y entra a /auth/login.
- Mostrar errores: INVALID_CREDENTIALS, ACCOUNT_DISABLED.
- Agregar ruta /auth/login con layout publico.
- Agregar indicador visual de sesion (nombre de usuario en topbar, boton de logout).
- Agregar tests unitarios de AuthService, AuthInterceptor y AuthGuard.

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

**Backend:**
- Crear UserAdminController bajo /api/admin/users.
- Implementar CreateInternalUserRequest y UpdateUserRequest.
- Validar regla branch_id segun rol.
- Aplicar @PreAuthorize para restringir ABM completo a ADMIN.
- Agregar tests de permisos y validacion branch_id.

**Frontend:**
- Crear modulo Angular admin/users con ruta /admin/users protegida por AdminGuard.
- Crear UserListComponent con tabla paginada (columnas: nombre, apellido, email, rol, sucursal, enabled, acciones).
- Crear UserFormComponent para alta y edicion de usuario interno.
- Validaciones en formulario: email formato, password >= 8 caracteres, branch_id obligatorio si rol es MANAGER o EMPLOYEE.
- Agregar selector de rol que muestre/oculte el campo branch_id segun corresponda.
- Agregar modal de confirmacion antes de habilitar/deshabilitar un usuario.
- Mostrar errores del backend: EMAIL_DUPLICATED, BRANCH_REQUIRED_FOR_ROLE.
- Manejar estados: loading (skeleton en tabla), empty (sin usuarios encontrados), error al cargar/guardar.
- Agregar tests unitarios de UserListComponent (listado, busqueda, paginacion) y UserFormComponent (creacion, edicion, validaciones).

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

**Backend:**
- Crear Category entity, repository, service y controller admin.
- Implementar DTOs CategoryRequest y CategoryDto.
- Implementar GET /api/store/categories.
- Agregar validacion de nombre obligatorio y parent existente.
- Agregar tests de creacion, edicion y parent invalido.

**Frontend (admin):**
- Crear CategoryListComponent con tabla paginada (columnas: nombre, categoria padre, descripcion, acciones).
- Crear CategoryFormComponent con campo nombre obligatorio, descripcion opcional y selector de categoria padre (tree/select anidado).
- Validar nombre obligatorio en tiempo real.
- Mostrar errores del backend: NAME_REQUIRED, PARENT_NOT_FOUND.
- Manejar estados: loading, empty (sin categorias), error.
- Agregar confirmacion antes de eliminar (con advertencia si tiene productos asociados).
- Agregar ruta /admin/categories dentro del layout admin.
- Agregar tests unitarios de CategoryListComponent y CategoryFormComponent.

**Frontend (publico):**
- Crear CategoryNavComponent para mostrar categorias en la tienda (sidebar o dropdown).
- Permitir seleccionar categoria para filtrar productos en el catalogo.

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

**Backend:**
- Crear Product entity, repository, service y admin controller.
- Implementar endpoints GET/POST/PUT/DELETE /api/admin/products.
- Crear ProductRequest, ProductSummaryDto y ProductDetailDto.
- Validar barcode unico, sale_price >= 0 y categoryId valido.
- Agregar tests de producto duplicado, precio invalido y edicion.

**Frontend:**
- Crear ProductListComponent con tabla paginada y filtros (busqueda por nombre/barcode, filtro por categoria, filtro por estado online).
- Crear ProductFormComponent con todos los campos: nombre, descripcion, marca, barcode, categoria (selector), precio de venta, stock minimo, imagen (upload con preview).
- Validaciones frontend: nombre obligatorio, precio >= 0, barcode formato opcional, categoria requerida.
- Implementar subida de imagen con preview (mostrar placeholder si no hay imagen).
- Mostrar estado online como badge de color (DRAFT=gris, PUBLISHED=verde, PAUSED=amarillo, HIDDEN=rojo).
- Manejar estados: loading (skeleton en tabla y formulario), empty (ningun producto), error al guardar/cargar.
- Agregar ruta /admin/products con listado, /admin/products/new y /admin/products/:id/edit.
- Agregar tests unitarios de ProductListComponent (filtros, paginacion) y ProductFormComponent (creacion, edicion, validaciones).

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

**Backend:**
- Crear OnlineStatus enum y transicion controlada en ProductService.
- Implementar PATCH /api/admin/products/{id}/status.
- Filtrar GET /api/store/products por PUBLISHED.
- Agregar tests para evitar estados invalidos y verificar filtro publico.

**Frontend:**
- Agregar en ProductListComponent un boton/cion por fila para cambiar estado (toggle menu con opciones disponibles segun estado actual).
- Crear StatusBadgeComponent reutilizable que muestre el estado con color y texto.
- Agregar modal de confirmacion al cambiar estado (ej: confirmar publicacion de producto, confirmar pausa, etc).
- Actualizar el badge visual inmediatamente despues del cambio sin recargar la tabla.
- Agregar tests unitarios del cambio de estado desde el frontend.

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

**Backend:**
- Implementar StoreProductController con listado y detalle.
- Crear queries paginadas por nombre, categoria y estado publicado.
- Agregar tests de endpoint publico y filtro por published.

**Frontend:**
- Crear StoreLayoutComponent (header con logo + nav + icono carrito + login/logout, footer).
- Crear StoreHomePageComponent con banner/bienvenida y productos destacados.
- Crear ProductGridComponent con grilla responsive (2 columnas mobile, 4 desktop).
- Crear ProductCardComponent con: imagen, nombre, precio, badge de stock, boton agregar al carrito.
- Crear ProductDetailPageComponent con: imagen grande, descripcion, precio, stock disponible, selector de cantidad, boton agregar.
- Crear StoreProductService con metodos getProducts(filters), getProduct(id), getCategories().
- Agregar buscador por nombre con debounce (300ms) en la cabecera de la tienda.
- Agregar filtro lateral de categorias con conteo de productos.
- Manejar estados visuales en cada componente: loading (skeleton cards), empty (mensaje sin resultados con sugerencia), error (alerta con reintentar).
- Agregar paginacion o infinite scroll en el listado de productos.
- Agregar ruta /store (home), /store/products (listado con filtros), /store/products/:id (detalle).
- Agregar tests unitarios de StoreProductService, ProductGridComponent, ProductCardComponent y ProductDetailPageComponent.

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

**Backend:**
- Crear ApiError record/clase.
- Crear DomainException base con code y status.
- Implementar @ControllerAdvice global.
- Mapear MethodArgumentNotValidException a VALIDATION_ERROR.
- Agregar tests de formato de error.

**Frontend:**
- Crear HttpErrorInterceptor que intercepte todas las respuestas HTTP con error.
- Mapear codigos de error del backend a mensajes amigables en espaniol.
- Crear ToastService para mostrar notificaciones de error/exito/info en esquina superior derecha.
- Crear ToastComponent global (snackbar) con duracion configurable y boton de cerrar.
- Manejar casos: 401 (redirigir a login), 403 (redirigir a home), 404 (mostrar no encontrado), 500 (mostrar error generico).
- Integrar validacion de formularios con los errores VALIDATION_ERROR del backend (mostrar error en el campo correspondiente).
- Crear ErrorPageComponent para rutas no encontradas (404) y errores de servidor (500).
- Agregar tests unitarios de HttpErrorInterceptor y ToastService.
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

**Backend / Testing:**
- Configurar JUnit, Mockito y Testcontainers PostgreSQL.
- Crear clase base para integration tests.
- Agregar script de seed demo con 15-20 productos representativos.
- Crear usuario admin, empleado y customer demo.
- Documentar credenciales demo no productivas.

**Frontend:**
- Configurar Jasmine y Karma para tests unitarios (o Jest si se prefiere).
- Crear test de ejemplo para un componente, un servicio y un guard.
- Agregar npm scripts: npm run test (unitarios), npm run test:watch, npm run lint, npm run build.
- Agregar datos de prueba (fixtures/mocks) para productos, categorias y usuario.
- Verificar que npm run build pase sin errores.
- Documentar comandos de testing frontend en README.
- Configurar proxy para que el frontend en desarrollo consuma la API con datos demo.

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

**Backend / DB:**
- Crear V4__inventory.sql con stock_lots y stock_movements.
- Crear StockLot, StockMovement y enums de movimientos.
- Crear indices por product_id, branch_id y expiration_date.
- Implementar StockLotRepository con consultas de disponibilidad.
- Implementar endpoint GET /api/admin/stock/lots.
- Agregar tests de calculo de stock disponible.

**Frontend:**
- No requiere subtareas frontend directas (es puramente modelo de datos).

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

**Backend:**
- Crear CreateStockLotRequest y StockLotDto.
- Implementar InventoryService.createStockLot().
- Registrar movimiento PURCHASE_ENTRY asociado al lote.
- Agregar tests de ingreso valido e invalido.

**Frontend:**
- Crear StockEntryPageComponent con formulario de carga de lote (producto selector, sucursal, cantidad, codigo de lote, vencimiento datepicker, costo opcional).
- Crear buscador/selector de producto por nombre o barcode.
- Validar cantidad > 0, vencimiento futuro (si se informa).
- Mostrar stock total actual del producto en la sucursal despues de cargar.
- Manejar estados: loading, error (producto no encontrado, cantidad invalida), exito con resumen.
- Agregar ruta /admin/stock/entry dentro del layout admin.
- Agregar tests unitarios del formulario y la validacion.

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

**Backend:**
- Crear FefoStockDeductionPolicy.
- Definir estructura DeductionPlan con lotId y quantityToDeduct.
- Implementar calculo sobre lista de lotes disponibles.
- Cubrir casos: un lote, varios lotes, lote sin vencimiento, stock insuficiente y cantidad decimal.
- Documentar criterio FEFO dentro del modulo inventory.

**Frontend:**
- No requiere subtareas frontend directas (es logica de dominio pura).

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

**Backend:**
- Crear StockAdjustmentRequest.
- Implementar validacion de motivo obligatorio.
- Implementar ajuste sobre lote especifico o producto/sucursal con FEFO para negativos.
- Registrar StockMovement con created_by_user_id.
- Crear listado de movimientos filtrable.
- Agregar tests de ajuste negativo sin stock y reason vacio.

**Frontend:**
- Crear StockAdjustmentPageComponent con formulario de ajuste (producto, sucursal, tipo MANUAL_ADJUSTMENT / INTERNAL_CONSUMPTION, cantidad positiva o negativa, motivo obligatorio).
- Mostrar stock actual del producto antes del ajuste.
- Validar motivo no vacio, cantidad distinta de cero.
- Crear StockMovementListComponent con tabla paginada y filtros (tipo de movimiento, producto, fecha).
- Mostrar badge de color por tipo de movimiento.
- Agregar ruta /admin/stock/adjustments y /admin/stock/movements.
- Agregar tests unitarios de formulario y listado.

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

**Backend:**
- Extender ProductSummaryDto y ProductDetailDto con availableStock.
- Optimizar consulta de disponibilidad para listado paginado.
- Validar branchId requerido o usar sucursal default del MVP.
- Agregar tests de producto publicado sin stock.

**Frontend:**
- Agregar indicador visual de stock en ProductCardComponent (badge verde si hay stock, rojo si agotado, gris si es bajo).
- Agregar detalle de stock disponible en ProductDetailPageComponent.
- Deshabilitar boton de agregar al carrito si stock = 0, mostrar mensaje de producto agotado.
- Agregar selector de sucursal (si aplica) o usar sucursal default configurada.
- Actualizar visualizacion de stock en tiempo real al cambiar de sucursal.
- Agregar tests de visualizacion de stock en ProductCardComponent.

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

**Backend / DB:**
- Crear V5__orders.sql con orders y order_items.
- Crear entidades Order y OrderItem.
- Crear enums OrderType, OrderStatus y FulfillmentType.
- Implementar OrderNumberGenerator.
- Crear repositorios y DTOs OrderSummaryDto/OrderDetailDto.
- Agregar constraints y tests de reglas ONLINE/POS.

**Frontend:**
- No requiere subtareas frontend directas (es puramente modelo de datos).

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

**Backend / DB:**
- Crear V6__payments.sql.
- Crear Payment entity y enums PaymentProvider, PaymentMethod, PaymentStatus.
- Crear PaymentRepository y PaymentService base.
- Agregar relacion Order -> Payments.
- Implementar consulta de payments por order.
- Agregar tests de constraints de metodo, proveedor y monto.

**Frontend:**
- No requiere subtareas frontend directas (es puramente modelo de datos).

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

**Backend:**
- Crear CustomerOrderController.
- Implementar CreateOnlineOrderRequest con items productId/quantity.
- Validar productos activos y publicados.
- Validar stock disponible por sucursal sin modificar lotes.
- Crear snapshots de cliente y producto en la orden.
- Calcular subtotal, descuentos en 0 y total.
- Agregar tests de stock insuficiente, producto inexistente y orden exitosa.

**Frontend:**
- El frontend de creacion de orden se cubre en S2-US10 (CheckoutPage).

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

**Frontend:**
- Crear CartService con Angular Signals para estado reactivo del carrito.
- Crear modelo CartItem con productId, name, price, imageUrl, quantity y availableStock.
- Persistir y recuperar carrito desde localStorage (serializar/deserializar).
- Implementar metodos: addItem(), removeItem(), updateQuantity(), clearCart(), getTotal(), getItemCount().
- Validar cantidad mayor a cero y no superior al disponible conocido.
- Crear CartDrawerComponent (sidebar deslizable desde la derecha) con lista de items, subtotal y boton de ir al checkout.
- Crear CartPageComponent (pagina completa de carrito) con tabla de items, modificar cantidades, eliminar, subtotal, boton de checkout.
- Agregar icono de carrito en el header de la tienda con badge de cantidad de items.
- Sincronizar el badge del carrito en el header cuando se agrega/quita un item.
- Agregar tests unitarios de CartService (agregar, quitar, persistencia, total) y CartDrawerComponent.

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

**Frontend:**
- Crear CheckoutPageComponent protegida por AuthGuard CUSTOMER.
- Crear CustomerOrderService con metodo createOrder(items).
- Mostrar resumen de compra: lista de items con foto, cantidad, precio unitario, subtotal por item, total general.
- Mostrar datos del cliente (email, nombre) desde el token JWT.
- Manejar error INSUFFICIENT_STOCK (mostrar cuales productos no tienen stock suficiente).
- Manejar error PRODUCT_NOT_FOUND / PRODUCT_NOT_PUBLISHED.
- Al crear la orden exitosamente, mostrar pantalla de confirmacion con numero de orden, estado PENDING_PAYMENT y resumen.
- NO vaciar el carrito automaticamente (esperar a que el usuario inicie pago en S3).
- Agregar boton de continuar comprando que redirija a la tienda.
- Agregar ruta /customer/checkout dentro del layout de tienda.
- Agregar tests unitarios de CheckoutPageComponent y CustomerOrderService.

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

**Backend:**
- Crear V3__suppliers.sql con suppliers y supplier_products.
- Crear entidades Supplier y SupplierProduct.
- Implementar endpoints /api/admin/suppliers.
- Implementar endpoints /api/admin/supplier-products.
- Agregar tests de CUIT duplicado, costo negativo y asociacion.

**Frontend:**
- Crear SupplierListComponent con tabla paginada (columnas: nombre, contacto, telefono, email, CUIT, acciones).
- Crear SupplierFormComponent para alta/edicion de proveedor.
- Validar CUIT formato, email formato, nombre obligatorio.
- Crear SupplierProductListComponent para ver productos asociados a un proveedor.
- Crear SupplierProductFormComponent para asociar producto a proveedor con costo actual y SKU del proveedor.
- Agregar buscador de producto para la asociacion.
- Manejar estados: loading, empty (sin proveedores), error.
- Agregar rutas /admin/suppliers y /admin/suppliers/:id/products.
- Agregar tests unitarios de componentes de proveedores.

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

**Backend:**
- Implementar endpoints de listado y detalle customer.
- Agregar validacion de ownership por customer_user_id.
- Agregar tests de acceso prohibido a pedido ajeno.

**Frontend:**
- Crear CustomerOrdersPageComponent con lista de pedidos del cliente (tabla: numero de orden, fecha, total, estado, acciones).
- Crear CustomerOrderDetailPageComponent con detalle completo: items (foto, nombre, cantidad, precio), total, estado actual, estado del pago, timeline de estados con fechas.
- Mostrar cada estado con badge de color y texto descriptivo.
- Agregar boton de ir al pago si el pedido esta PENDING_PAYMENT (prepara para S3-US05).
- Manejar estados: loading, empty (ningun pedido aun), error.
- Agregar ruta /customer/orders y /customer/orders/:id.
- Agregar tests unitarios de CustomerOrdersPageComponent y CustomerOrderDetailPageComponent.

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

**Backend:**
- Crear interfaz PaymentGateway.
- Crear DTO interno PaymentPreferenceResult con initPoint y preferenceId.
- Crear MercadoPagoGateway con configuracion externa.
- Crear FakePaymentGateway para tests/dev.
- Agregar properties MP_ACCESS_TOKEN, MP_WEBHOOK_SECRET, MP_SUCCESS_URL, MP_FAILURE_URL.
- Documentar como usar sandbox/mock.

**Frontend:**
- No requiere subtareas frontend directas (es infraestructura backend).

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

**Backend:**
- Crear MercadoPagoService.createCheckoutPreference().
- Validar estado de orden PENDING_PAYMENT.
- Validar ownership CUSTOMER.
- Guardar provider_preference_id y external_reference en payment.
- Mapear datos de order/items al request de MP.
- Agregar tests de preferencia nueva, preferencia existente y orden invalida.

**Frontend:**
- El frontend de pago se cubre en S3-US05 (integracion checkout MP).

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

**Backend:**
- Crear endpoint publico MercadoPagoWebhookController.
- Implementar verificacion de firma en gateway.
- Implementar busqueda por provider_payment_id/provider_preference_id/external_reference.
- Agregar control idempotente antes de modificar orden o stock.
- Mapear estados MP a PaymentStatus interno.
- Registrar metadata JSONB del webhook sin datos sensibles.
- Agregar integration tests de webhook duplicado y rechazado.

**Frontend:**
- No requiere subtareas frontend directas (es procesamiento asincronico server-side).

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

**Backend:**
- Implementar InventoryService.deductForOnlineOrder() transaccional.
- Crear consulta repository con bloqueo pesimista ordenada por vencimiento.
- Registrar StockMovement por lote afectado.
- Actualizar paid_at y approved_at.
- Implementar manejo STOCK_CONFLICT.
- Agregar tests concurrentes basicos de no sobreventa.
- Agregar tests de descuento de varios lotes.

**Frontend:**
- No requiere subtareas frontend directas (es logica de backend transaccional).

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

**Frontend:**
- Crear CustomerCheckoutService con metodo requestCheckout(orderId) que llame a POST /api/customer/orders/{id}/checkout/mp.
- Agregar boton de Pagar con Mercado Pago en la pantalla de detalle de orden (CustomerOrderDetailPage) cuando la orden esta PENDING_PAYMENT.
- Al hacer clic, llamar al endpoint y redirigir al navegador a initPoint con window.location.href.
- Crear PaymentCallbackPageComponent para manejar el retorno desde MP (success, failure, pending).
- En PaymentCallbackPage, consultar GET /api/customer/orders/{id} para obtener el estado actualizado.
- Manejar estado asincronico: si el webhook aun no impacto, mostrar mensaje de pago pendiente con instrucciones.
- Mostrar pantalla de exito cuando order.status = PAID (con resumen y numero de orden).
- Mostrar pantalla de rechazo cuando order.status = PAYMENT_FAILED (con posibilidad de reintentar).
- Mostrar pantalla de STOCK_CONFLICT con mensaje de contacto al local.
- Agregar ruta /customer/payment/callback con query params orderId.
- Agregar tests unitarios de CustomerCheckoutService y PaymentCallbackPageComponent.

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

**Backend:**
- Crear V7__cash.sql con cash_sessions y cash_movements.
- Crear CashSession entity, repository y status enum.
- Implementar CashService.openCashSession().
- Validar una caja abierta por sucursal.
- Crear endpoints open/current.
- Agregar tests de caja duplicada y apertura por empleado.

**Frontend:**
- Crear CashOpenPageComponent con formulario de apertura (monto inicial obligatorio, notas opcional, sucursal del usuario).
- Mostrar indicador visual en el sidebar del admin cuando hay una caja abierta.
- Si ya hay una caja abierta, mostrar mensaje y redirigir a la caja actual.
- Redirigir al detalle de caja actual despues de abrir.
- Manejar errores: CASH_SESSION_ALREADY_OPEN.
- Agregar ruta /admin/cash/open.
- Agregar tests unitarios del formulario de apertura.

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

**Backend:**
- Crear CashMovement entity y enums.
- Implementar POST /api/admin/cash-sessions/{id}/movements.
- Validar reason obligatorio y amount distinto de cero.
- Actualizar detalle de caja con movimientos manuales.
- Agregar tests de caja cerrada, reason vacio y usuario creador.

**Frontend:**
- Crear CashMovementFormComponent con formulario (tipo: CASH_IN / CASH_OUT / ADJUSTMENT, metodo: CASH / TRANSFER / OTHER, monto, motivo obligatorio).
- Mostrar el formulario en la pagina de detalle de caja actual.
- Validar monto distinto de cero, motivo no vacio.
- Deshabilitar formulario si la caja esta CLOSED.
- Agregar tabla de movimientos de la caja actual con badge de color por tipo.
- Agregar tests unitarios del formulario y validaciones.

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

**Backend:**
- Crear CashCloseCalculator.
- Implementar POST /api/admin/cash-sessions/{id}/close.
- Calcular totalsByMethod desde payments asociados a la caja.
- Persistir counted, expected, difference, reason, closed_by_user_id y closed_at.
- Agregar tests de cierre sin diferencia, con diferencia sin razon y con diferencia justificada.

**Frontend:**
- Crear CashClosePageComponent con resumen de cierre: apertura, totales por metodo de pago (CASH, QR, TRANSFER, DEBIT_CARD, CREDIT_CARD), total de movimientos manuales, expected cash amount.
- Agregar campo de efectivo contado (countedCashAmount) que el empleado ingresa.
- Calcular y mostrar automaticamente la diferencia (expected - counted).
- Si la diferencia es distinta de cero, mostrar campo obligatorio de motivo de diferencia.
- Confirmar cierre con modal de confirmacion mostrando el resumen completo.
- Despues del cierre, mostrar pantalla de resumen final con todos los datos.
- Agregar ruta /admin/cash/close/:sessionId.
- Agregar tests unitarios del componente de cierre (calculo de diferencia, validacion de motivo).

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

**Backend:**
- Crear endpoint especifico o reutilizar catalogo admin con filtro q/barcode.
- Optimizar indice de barcode.
- Agregar tests de busqueda por barcode.

**Frontend:**
- Crear POSProductSearchComponent con input de busqueda que filtre en tiempo real (nombre y barcode).
- Enfocar input automaticamente al cargar, ideal para lector de codigo de barras como entrada de teclado.
- Mostrar resultados como cards compactas con: nombre, precio, stock disponible, barcode.
- Detectar si el input es un barcode (solo numeros) o nombre para optimizar busqueda.
- Agregar indicador visual de producto sin stock (deshabilitado).
- Al seleccionar un producto, agregarlo al carrito POS y limpiar el input de busqueda para el siguiente producto.
- Soporte para lectura continua de codigos de barras (enter rapido).
- Agregar tests unitarios del componente de busqueda.

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

**Backend:**
- Implementar PosSaleService.createSale() con @Transactional.
- Validar caja OPEN de la sucursal del usuario.
- Bloquear lotes con FOR UPDATE y aplicar FEFO.
- Crear order POS con status COMPLETED.
- Crear payment provider MANUAL segun metodo.
- Crear snapshots de items.
- Registrar movimientos POS_SALE por lote.
- Agregar integration tests de venta cash, QR, stock insuficiente y sin caja.

**Frontend:**
- El frontend de venta POS se cubre en S3-US11 (pantalla POS completa).

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

**Frontend:**
- Crear AdminPosPageComponent como pantalla completa de venta presencial.
- Dividir en dos paneles: izquierdo (busqueda + resultados) y derecho (carrito POS).
- Integrar POSProductSearchComponent en el panel izquierdo.
- Crear POSCartComponent en el panel derecho con: tabla de items agregados (nombre, cantidad, precio, subtotal, boton quitar), modificador de cantidad, total general.
- Implementar selector de metodo de pago (CASH, QR, TRANSFER, DEBIT_CARD, CREDIT_CARD) con iconos.
- Validar que haya caja abierta antes de permitir cobrar (mostrar error si no).
- Al presionar cobrar, consumir POST /api/admin/pos/sales.
- Mostrar resultado: resumen de venta con numero de orden y metodo de pago.
- Manejar errores: CASH_SESSION_REQUIRED, INSUFFICIENT_STOCK.
- Agregar boton de nueva venta que limpie el carrito POS.
- Detectar tecla F8 o similar para cobrar rapido.
- Agregar tests unitarios del AdminPosPageComponent y POSCartComponent.

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

**Backend:**
- Crear fixtures de productos, lotes, usuarios y caja.
- Testear webhook aprobado con descuento FEFO.
- Testear webhook duplicado sin doble descuento.
- Testear POS con caja abierta y sin caja.
- Testear cierre de caja con diferencia y razon obligatoria.
- Agregar comando unico backend test.

**Frontend:**
- Agregar tests de integracion del flujo completo de checkout + MP mock.
- Verificar que los componentes POS, caja y checkout manejen correctamente los estados de error del backend.

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

**Backend:**
- Implementar PATCH /api/admin/orders/{id}/prepare.
- Implementar PATCH /api/admin/orders/{id}/ready.
- Implementar PATCH /api/admin/orders/{id}/delivered.
- Validar transiciones con OrderStatePolicy.
- Agregar tests de transiciones validas e invalidas.

**Frontend:**
- Agregar botones de accion por estado en AdminOrderDetailPage (preparar, listo, entregado) que aparecen segun el estado actual.
- Deshabilitar botones que no correspondan segun la maquina de estados.
- Agregar modal de confirmacion antes de cada transicion de estado.
- Actualizar el estado visualmente sin recargar la pagina despues de la accion.
- Mostrar timeline de estados con timestamps en el detalle de la orden.
- Agregar tests unitarios de los botones de transicion y la logica de visibilidad.

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

**Backend:**
- Implementar OrderCancellationService transaccional.
- Buscar stock_movements originales por order_id.
- Restaurar quantity_available en los mismos stock_lot_id.
- Crear movimientos inversos CANCELLATION_RETURN.
- Actualizar status CANCELLED y cancellation_reason.
- Actualizar payment CANCELLED o REFUNDED segun escenario.
- Agregar tests de cancelacion PAID, PREPARING, READY y estado invalido.

**Frontend:**
- Agregar boton de Cancelar orden en AdminOrderDetailPage (visible solo para estados cancelables).
- Crear modal de cancelacion con campo de motivo obligatorio.
- Validar motivo no vacio antes de enviar.
- Mostrar confirmacion con resumen de la orden antes de cancelar.
- Actualizar estado visualmente despues de la cancelacion.
- Agregar tests unitarios del modal de cancelacion.

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

**Backend:**
- Implementar filtros de OrderRepository.
- Agregar tests de filtros y permisos por rol/sucursal.

**Frontend:**
- Crear AdminOrdersPageComponent con tabla paginada y filtros (estado, tipo ONLINE/POS, rango de fechas, busqueda por numero de orden).
- Crear AdminOrderDetailPageComponent con: datos del cliente (snapshot), items con foto, pagos asociados, total, estado actual, timeline de estados.
- Mostrar acciones disponibles segun estado: preparar, listo para retirar, entregado, cancelar.
- Filtrar ordenes por sucursal del usuario logueado (MANAGER/EMPLOYEE ven solo su sucursal, ADMIN ve todas).
- Manejar estados: loading, empty (sin ordenes con esos filtros), error.
- Agregar ruta /admin/orders y /admin/orders/:id.
- Agregar tests unitarios de AdminOrdersPageComponent (filtros, paginacion, permisos por sucursal).

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

**Backend:**
- Crear ReportService.dashboard().
- Crear queries agregadas para ventas por tipo y dia.
- Crear query de productos mas vendidos.
- Crear query de pedidos pendientes.
- Agregar tests de agregacion por sucursal y rol.

**Frontend:**
- Crear AdminDashboardPageComponent como pagina principal del backoffice.
- Crear DashboardCardComponent reutilizable (icono, titulo, valor, color, link opcional).
- Mostrar cards de: ventas del dia (online + POS), ordenes pendientes, productos con stock bajo, lotes por vencer.
- Crear tabla de productos top (mas vendidos del dia/semana) con posicion, nombre, cantidad, imagen pequena.
- Filtrar datos segun el rol (si es MANAGER/EMPLOYEE, solo su sucursal).
- Agregar skeleton loading mientras cargan las cards.
- Mostrar estado empty si no hay datos del dia.
- Actualizar periodicamente o con boton de refrescar.
- Agregar ruta /admin/dashboard como pagina de inicio del admin layout.
- Agregar tests unitarios de AdminDashboardPageComponent y DashboardCardComponent.

---

## S4-US05 -- Generar reporte de cierre de caja

**Epica:** EP-11 | **Prioridad:** Media | **Points:** 5

**Como** administrador, **quiero** ver detalle de un cierre con totales por medio de pago **para** auditar caja y entender diferencias de efectivo.

**Criterios de aceptacion:**
- GET /api/admin/reports/cash-session/{id} devuelve sesion, pagos, totales por metodo, efectivo esperado, contado, diferencia y razon.
- El reporte separa arqueo de efectivo de totales informativos no cash.
- La UI permite consultar historial y detalle de cierres.

**Subtareas sugeridas:**

**Backend:**
- Implementar CashReportDto.
- Crear query de totalsByMethod desde payments.
- Agregar tests de totales por CASH/QR/TRANSFER/TARJETAS.

**Frontend:**
- Crear CashSessionHistoryPageComponent con lista de cierres anteriores (tabla: fecha apertura, fecha cierre, operador, expected, counted, diferencia, estado).
- Crear CashSessionDetailReportPageComponent con detalle completo de un cierre: datos de apertura, totales por metodo de pago, movimientos manuales, expected cash amount, counted, diferencia y motivo.
- Mostrar la diferencia con color verde (0 o positiva) o rojo (negativa), y el motivo si existe.
- Manejar estados: loading, empty (sin cierres), error.
- Agregar rutas /admin/cash/history y /admin/cash/history/:sessionId.
- Agregar tests unitarios de ambos componentes.

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

**Backend:**
- Crear RecommendationService rule-based.
- Implementar regla LOW_STOCK usando products.minimum_stock y stock disponible.
- Implementar regla EXPIRING_SOON con lotes proximos a vencer.
- Implementar regla HIGH_ROTATION con order_items recientes.
- Implementar regla NO_MOVEMENT con productos sin ventas recientes.
- Agregar tests de cada regla con datos controlados.

**Frontend:**
- Crear RecommendationsPanelComponent con lista de recomendaciones agrupadas por tipo.
- Mostrar cada recomendacion con: icono segun tipo, titulo, descripcion, urgencia (alta/media/baja con color), link a la accion correspondiente.
- Agregar reglas de visualizacion: ocultar panel si no hay recomendaciones.
- Integrar en el dashboard principal (AdminDashboardPage) como seccion secundaria.
- Agregar ruta /admin/recommendations para ver todas.
- Agregar tests unitarios del panel de recomendaciones.

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

**Backend:**
- Crear V9__audit.sql y AuditLog entity.
- Crear AuditService.record().
- Integrar auditoria en ProductService para cambios de precio.
- Integrar auditoria en InventoryService para ajustes.
- Integrar auditoria en CashService para apertura/cierre/movimientos.
- Integrar auditoria en Payment/Webhook para confirmaciones.
- Crear endpoint GET /api/admin/audit-logs con filtros basicos.
- Agregar tests de generacion de audit logs.

**Frontend:**
- Crear AuditLogListPageComponent con tabla paginada de logs (fecha, usuario, accion, entidad, descripcion).
- Agregar filtros: por entidad, por usuario, rango de fechas.
- Mostrar detalle expandible de cada log.
- Manejar estados: loading, empty (sin logs), error.
- Agregar ruta /admin/audit accessible solo por ADMIN.
- Agregar tests unitarios del listado de auditoria.

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

**Backend:**
- Revisar @PreAuthorize en todos los controllers.
- Crear util de seguridad para branch scope.
- Validar que los endpoints customer no expongan pedidos ajenos.
- Agregar tests de acceso prohibido por rol.

**Frontend:**
- Completar AdminGuard (redirige a /store si el rol no es ADMIN/MANAGER/EMPLOYEE).
- Completar CustomerGuard (redirige a /auth/login si el rol no es CUSTOMER).
- Implementar menu dinamico en AdminLayout segun rol: ADMIN ve todo, MANAGER no ve auditoria ni ABM de usuarios, EMPLOYEE solo ve POS, caja y pedidos.
- Ocultar rutas completas en el sidebar segun rol (no solo deshabilitar).
- Redirigir a dashboard si el usuario entra a una ruta no permitida.
- Agregar tests de guard y visibilidad de menu por rol.

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

**Frontend:**
- Revisar layout publico de tienda en mobile (StoreLayout: menu hamburguesa, grilla de 2 columnas, carrito drawer a pantalla completa).
- Revisar layout backoffice (AdminLayout: sidebar colapsable en mobile, topbar compacta).
- Agregar estados empty/loading/error consistentes en todas las pantallas (usar componentes compartidos EmptyStateComponent, LoadingSkeletonComponent, ErrorAlertComponent).
- Optimizar formularios de stock, caja y POS para uso tactil.
- Agregar confirmaciones para acciones destructivas (cancelar orden, eliminar producto, cerrar caja con diferencia).
- Agregar atajos de teclado basicos en POS (F8 para cobrar, Escape para limpiar busqueda).
- Revisar contraste de colores y tamanios de fuente minimos.
- Probar en resoluciones: 1366x768 (notebook), 1920x1080 (escritorio), 375x667 (mobile), 768x1024 (tablet).
- Corregir problemas de accesibilidad: labels en formularios, roles ARIA en componentes interactivos, foco visible.

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

**Frontend / QA:**
- Definir herramienta E2E: configurar Playwright o Cypress en el proyecto frontend.
- Crear dataset estable de demo con productos, stock y usuarios conocidos.
- Crear test E2E del flujo customer: registro en /auth/register, navegar catalogo en /store, buscar producto, agregar al carrito, ir al checkout, confirmar orden.
- Crear test E2E del flujo empleado: login como EMPLOYEE, abrir caja, buscar producto en POS, vender con metodo CASH, cerrar caja.
- Crear test E2E del flujo admin: login como ADMIN, ver dashboard, navegar a productos, crear producto, ver ordenes, cancelar orden.
- Documentar bugs encontrados y correcciones aplicadas.
- Agregar script npm run e2e para ejecutar todos los tests.

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

**Backend / Infra:**
- Crear Dockerfile backend con build multi-stage.
- Configurar CORS para ambiente demo.
- Configurar variables de Mercado Pago sandbox/mock.
- Crear script de migracion/seed para demo.
- Documentar pasos de despliegue y rollback simple.

**Frontend:**
- Configurar build de produccion con optimizaciones (--prod, --aot, --optimization).
- Configurar servidor estatico (Nginx) para servir el frontend build.
- Configurar proxy reverso en Nginx para redirigir /api/ al backend.
- Validar demo desde navegador limpio (incognito, sin cache).

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

**Documentacion:**
- Actualizar README funcional y tecnico.
- Actualizar documentacion de endpoints implementados.
- Agregar diagrama simple de arquitectura final.
- Preparar guion de demo: customer, empleado, admin.
- Adjuntar capturas de: catalogo publico, detalle de producto, carrito, checkout, pago (mock), pantalla de orden creada, listado de pedidos del cliente, POS con productos agregados, apertura de caja, cierre de caja con resumen, dashboard, panel de recomendaciones, ABM de productos, listado de ordenes admin, y auditoria.
- Revisar que Jira tenga dependencias, story points y criterios de aceptacion completos.

---

> [!seealso] Notas relacionadas
> - [[Epicas]]
> - [[MVP]]
> - [[Roadmap]]
> - Volver a [[_Index]]
