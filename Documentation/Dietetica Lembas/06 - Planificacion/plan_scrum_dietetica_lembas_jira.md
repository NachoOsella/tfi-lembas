# Plan Scrum detallado — Dietética Lembas MVP

> Fuente base: documentación actualizada `docu.zip` del proyecto Dietética Lembas. Este plan respeta el alcance MVP: monolito modular Spring Boot, Angular por features, PostgreSQL/Flyway, tienda online con Mercado Pago Checkout Pro, caja operativa, venta POS, stock por lotes FEFO, pagos unificados, roles `ADMIN/MANAGER/EMPLOYEE/CUSTOMER` y solo retiro en sucursal.

## Convenciones para cargar en Jira

- **Duración:** 4 sprints de 2 semanas.
- **Issue types sugeridos:** Epic → Story → Sub-task.
- **Puntos:** estimación relativa para balancear complejidad, no horas exactas.
- **Prioridades:** Alta = bloqueante o flujo crítico; Media = necesaria para MVP pero no bloquea la base; Baja = opcional/postergable.
- **Definition of Done general:** código implementado, validaciones backend/frontend, tests mínimos, sin errores críticos, migraciones aplicadas, permisos revisados y documentación actualizada cuando corresponda.

## Épicas del proyecto

| Key | Épica | Descripción |
|---|---|---|
| EP-00 | Plataforma técnica, calidad y despliegue | Base transversal del proyecto: estructura, Docker, perfiles, errores, testing, CI/CD, datos demo, documentación y despliegue. |
| EP-01 | Autenticación y registro | Registro CUSTOMER, login, JWT, BCrypt, sesión frontend y seguridad base. |
| EP-02 | Gestión de catálogo | Categorías, productos, precios, códigos de barras, imagen única y estado online. |
| EP-03 | Tienda online | Catálogo público, carrito local, checkout autenticado, consulta de pedidos propios y resultado de compra. |
| EP-04 | Mercado Pago Checkout Pro | Adaptador de pago, creación de preferencia, webhook, idempotencia y estados de pago. |
| EP-05 | Stock | Stock por lotes y sucursal, movimientos, FEFO, ajustes, alertas de vencimiento y bajo stock. |
| EP-06 | Pedidos | Órdenes unificadas POS/ONLINE, items con snapshot, máquina de estados, preparación, entrega y cancelación. |
| EP-07 | Caja operativa | Apertura, caja actual, movimientos manuales, cierre, arqueo de efectivo y diferencia justificada. |
| EP-08 | Ventas presenciales POS | Venta rápida con búsqueda/código de barras, caja abierta obligatoria, cobro y descuento FEFO. |
| EP-09 | Pagos unificados | Tabla payments común para pagos online y presenciales, métodos, estados y trazabilidad. |
| EP-10 | Proveedores | ABM de proveedores, asociación producto-proveedor y costo manual actual. |
| EP-11 | Reportes y recomendaciones | Dashboard, reporte de caja, recomendaciones por reglas, stock bajo, vencimientos y rotación. |
| EP-12 | Usuarios y roles | Usuarios internos, asignación de sucursal, RBAC por rol y visibilidad de interfaz según permisos. |

## Balance general de sprints

| Sprint | Objetivo | Stories | Story points |
|---|---|---:|---:|
| Sprint 1 | Construir la base técnica y de dominio mínima: entorno, migraciones core, autenticación, roles, catálogo administrable y catálogo público inicial. | 12 | 75 |
| Sprint 2 | Incorporar inventario real por lotes, FEFO, órdenes online pendientes de pago, carrito local, proveedores y consulta de pedidos del cliente. | 12 | 75 |
| Sprint 3 | Completar los flujos transaccionales principales: Mercado Pago, webhook, descuento FEFO online, caja operativa y venta presencial POS. | 12 | 75 |
| Sprint 4 | Cerrar operación backoffice y calidad final: estados de pedidos, cancelaciones, reportes, recomendaciones, auditoría, seguridad, E2E, demo y documentación. | 12 | 75 |

# Sprint 1 — 2 semanas

**Objetivo del sprint:** Construir la base técnica y de dominio mínima: entorno, migraciones core, autenticación, roles, catálogo administrable y catálogo público inicial.

**Entregables esperados:**
- Backend y frontend ejecutables localmente.
- Base PostgreSQL con migraciones core/catalog y seed mínimo.
- Registro/login CUSTOMER e inicio de seguridad JWT.
- Usuarios internos con roles fijos.
- ABM de categorías/productos y catálogo público publicado.

| Story | Épica | Título | Prioridad | Puntos | Dependencias |
|---|---|---|---|---:|---|
| S1-US01 | EP-00 | Preparar estructura base del proyecto | Alta | 5 | - |
| S1-US02 | EP-00 | Configurar ambiente local con Docker Compose | Alta | 5 | S1-US01 |
| S1-US03 | EP-00 | Implementar migraciones Flyway iniciales | Alta | 8 | S1-US02 |
| S1-US04 | EP-01 | Registrar clientes CUSTOMER | Alta | 5 | S1-US03 |
| S1-US05 | EP-01 | Iniciar sesión con JWT | Alta | 8 | S1-US04 |
| S1-US06 | EP-12 | Gestionar usuarios internos y roles | Alta | 8 | S1-US05 |
| S1-US07 | EP-02 | Administrar categorías de productos | Media | 5 | S1-US03 |
| S1-US08 | EP-02 | Administrar productos del catálogo | Alta | 8 | S1-US07 |
| S1-US09 | EP-02 | Publicar y pausar productos en tienda | Media | 5 | S1-US08 |
| S1-US10 | EP-03 | Mostrar catálogo público inicial | Alta | 8 | S1-US09 |
| S1-US11 | EP-00 | Estandarizar errores y validaciones de API | Alta | 5 | S1-US04 |
| S1-US12 | EP-00 | Dejar base de testing y datos demo | Media | 5 | S1-US03 |

## S1-US01 — Preparar estructura base del proyecto

**Épica:** EP-00  
**Sprint:** Sprint 1  
**Prioridad:** Alta  
**Story points:** 5  
**Dependencias:** -  
**Labels:** `setup,arquitectura`

**Historia de usuario:** Como desarrollador, quiero tener el proyecto organizado como monolito modular con frontend separado para poder desarrollar módulos sin mezclar responsabilidades desde el inicio.

**Criterios de aceptación:**
- Existe estructura backend por módulos: auth, users, catalog, inventory, orders, payments, cash, suppliers, reports, audit y shared.
- Existe estructura frontend Angular por features: store, customer, admin, auth y shared.
- El proyecto levanta en local con configuración reproducible.
- La estructura queda documentada en README técnico.

**Subtareas técnicas sugeridas:**
- [S1-US01-T01] Crear repositorio/estructura con carpetas backend, frontend, infra y docs.
- [S1-US01-T02] Crear paquete shared para DTOs comunes, excepciones y utilidades.
- [S1-US01-T03] Configurar perfiles dev/test en backend y environment files en Angular.
- [S1-US01-T04] Agregar README con comandos de ejecución, estructura y convenciones de ramas.
- [S1-US01-T05] Definir convención de nombres para endpoints, DTOs, servicios y componentes.

## S1-US02 — Configurar ambiente local con Docker Compose

**Épica:** EP-00  
**Sprint:** Sprint 1  
**Prioridad:** Alta  
**Story points:** 5  
**Dependencias:** S1-US01  
**Labels:** `setup,docker,postgres`

**Historia de usuario:** Como desarrollador, quiero levantar PostgreSQL y servicios de la aplicación con un comando para evitar instalaciones manuales y facilitar la demo académica.

**Criterios de aceptación:**
- Docker Compose levanta PostgreSQL 16 con volumen persistente.
- El backend se conecta a la base usando variables de entorno.
- El frontend puede ejecutarse localmente consumiendo la API.
- Hay archivo .env.example sin credenciales reales.

**Subtareas técnicas sugeridas:**
- [S1-US02-T01] Crear docker-compose.yml con PostgreSQL 16 y red interna.
- [S1-US02-T02] Definir variables DB_HOST, DB_PORT, DB_NAME, DB_USER, DB_PASSWORD y JWT_SECRET.
- [S1-US02-T03] Configurar application-dev.yml y application-test.yml.
- [S1-US02-T04] Agregar script de reset de base local para desarrollo.
- [S1-US02-T05] Documentar comandos: up, down, logs, migrate y seed.

## S1-US03 — Implementar migraciones Flyway iniciales

**Épica:** EP-00  
**Sprint:** Sprint 1  
**Prioridad:** Alta  
**Story points:** 8  
**Dependencias:** S1-US02  
**Labels:** `db,flyway,core`

**Historia de usuario:** Como desarrollador, quiero versionar el esquema de base de datos desde el primer sprint para mantener trazabilidad y reproducibilidad del modelo.

**Criterios de aceptación:**
- Flyway ejecuta migraciones en orden al iniciar el backend.
- Existen migraciones para branches, users, categories y products.
- Las constraints principales de roles, precios y estados online están en base.
- Hay seed inicial mínimo para operar la demo.

**Subtareas técnicas sugeridas:**
- [S1-US03-T01] Configurar dependencia Flyway en Spring Boot.
- [S1-US03-T02] Crear V1__core.sql con branches y users.
- [S1-US03-T03] Crear V2__catalog.sql con categories y products.
- [S1-US03-T04] Agregar CHECK para role ADMIN/MANAGER/EMPLOYEE/CUSTOMER.
- [S1-US03-T05] Agregar CHECK para online_status DRAFT/PUBLISHED/PAUSED/HIDDEN.
- [S1-US03-T06] Crear V10__seed_data.sql parcial con sucursal Centro, admin demo y categorías base.

## S1-US04 — Registrar clientes CUSTOMER

**Épica:** EP-01  
**Sprint:** Sprint 1  
**Prioridad:** Alta  
**Story points:** 5  
**Dependencias:** S1-US03  
**Labels:** `auth,customer,security`

**Historia de usuario:** Como cliente, quiero registrarme con email y contraseña para poder comprar online con una cuenta propia.

**Criterios de aceptación:**
- El endpoint POST /api/auth/register crea usuarios con rol CUSTOMER.
- El email es único y se devuelve EMAIL_DUPLICATED ante duplicado.
- La contraseña se almacena hasheada con BCrypt.
- CUSTOMER queda con branch_id null.

**Subtareas técnicas sugeridas:**
- [S1-US04-T01] Crear User entity, UserRepository y Role enum.
- [S1-US04-T02] Crear RegisterRequest, AuthResponse y UserDto.
- [S1-US04-T03] Implementar AuthService.registerCustomer().
- [S1-US04-T04] Validar email, password, firstName, lastName y phone opcional.
- [S1-US04-T05] Crear tests unitarios de registro exitoso, email duplicado y password hash.

## S1-US05 — Iniciar sesión con JWT

**Épica:** EP-01  
**Sprint:** Sprint 1  
**Prioridad:** Alta  
**Story points:** 8  
**Dependencias:** S1-US04  
**Labels:** `auth,jwt,security`

**Historia de usuario:** Como usuario, quiero iniciar sesión con email y contraseña para acceder a rutas protegidas según mi rol.

**Criterios de aceptación:**
- POST /api/auth/login devuelve token JWT y datos del usuario.
- Credenciales inválidas devuelven INVALID_CREDENTIALS.
- Usuarios disabled no pueden iniciar sesión.
- GET /api/auth/me devuelve usuario autenticado desde el token.

**Subtareas técnicas sugeridas:**
- [S1-US05-T01] Configurar Spring Security stateless y BCryptPasswordEncoder.
- [S1-US05-T02] Implementar JwtService para generar y validar token de 24h.
- [S1-US05-T03] Implementar JwtAuthenticationFilter.
- [S1-US05-T04] Configurar rutas públicas /api/auth/**, /api/store/**, /api/webhooks/** y /uploads/**.
- [S1-US05-T05] Crear AuthController con login y me.
- [S1-US05-T06] Agregar tests de login, token inválido y usuario deshabilitado.

## S1-US06 — Gestionar usuarios internos y roles

**Épica:** EP-12  
**Sprint:** Sprint 1  
**Prioridad:** Alta  
**Story points:** 8  
**Dependencias:** S1-US05  
**Labels:** `users,roles,rbac`

**Historia de usuario:** Como administrador, quiero crear y administrar usuarios internos para controlar quién opera backoffice, caja, stock y pedidos.

**Criterios de aceptación:**
- ADMIN puede listar, crear, editar, habilitar y deshabilitar usuarios internos.
- MANAGER y EMPLOYEE requieren branch_id obligatorio.
- CUSTOMER no se crea desde el ABM interno.
- Las acciones críticas de usuario quedan preparadas para auditoría.

**Subtareas técnicas sugeridas:**
- [S1-US06-T01] Crear UserAdminController bajo /api/admin/users.
- [S1-US06-T02] Implementar CreateInternalUserRequest y UpdateUserRequest.
- [S1-US06-T03] Validar regla branch_id según rol.
- [S1-US06-T04] Aplicar @PreAuthorize para restringir ABM completo a ADMIN.
- [S1-US06-T05] Crear pantalla Angular de listado básico de usuarios internos.
- [S1-US06-T06] Agregar tests de permisos y validación branch_id.

## S1-US07 — Administrar categorías de productos

**Épica:** EP-02  
**Sprint:** Sprint 1  
**Prioridad:** Media  
**Story points:** 5  
**Dependencias:** S1-US03  
**Labels:** `catalog,categories`

**Historia de usuario:** Como administrador, quiero crear y editar categorías para organizar el catálogo y permitir filtros en la tienda.

**Criterios de aceptación:**
- ADMIN/MANAGER pueden crear, editar y listar categorías.
- Las categorías pueden tener parent_id opcional.
- La tienda pública puede consultar categorías activas con conteo de productos cuando aplique.
- No se permite eliminar una categoría usada sin manejo controlado.

**Subtareas técnicas sugeridas:**
- [S1-US07-T01] Crear Category entity, repository, service y controller admin.
- [S1-US07-T02] Implementar DTOs CategoryRequest y CategoryDto.
- [S1-US07-T03] Implementar GET /api/store/categories.
- [S1-US07-T04] Agregar validación de nombre obligatorio y parent existente.
- [S1-US07-T05] Crear componentes Angular de listado/formulario simple.
- [S1-US07-T06] Agregar tests de creación, edición y parent inválido.

## S1-US08 — Administrar productos del catálogo

**Épica:** EP-02  
**Sprint:** Sprint 1  
**Prioridad:** Alta  
**Story points:** 8  
**Dependencias:** S1-US07  
**Labels:** `catalog,products`

**Historia de usuario:** Como administrador, quiero crear y editar productos con precio, categoría, marca, imagen y código de barras para mantener actualizado el catálogo comercial.

**Criterios de aceptación:**
- ADMIN/MANAGER pueden crear, editar, listar y desactivar productos.
- El barcode es único cuando se informa.
- El precio de venta no puede ser negativo.
- El producto guarda online_status para controlar visibilidad pública.

**Subtareas técnicas sugeridas:**
- [S1-US08-T01] Crear Product entity, repository, service y admin controller.
- [S1-US08-T02] Implementar endpoints GET/POST/PUT/DELETE /api/admin/products.
- [S1-US08-T03] Crear ProductRequest, ProductSummaryDto y ProductDetailDto.
- [S1-US08-T04] Validar barcode único, sale_price >= 0 y categoryId válido.
- [S1-US08-T05] Crear pantalla Angular admin de productos con tabla paginada.
- [S1-US08-T06] Crear formulario de alta/edición con validaciones frontend.
- [S1-US08-T07] Agregar tests de producto duplicado, precio inválido y edición.

## S1-US09 — Publicar y pausar productos en tienda

**Épica:** EP-02  
**Sprint:** Sprint 1  
**Prioridad:** Media  
**Story points:** 5  
**Dependencias:** S1-US08  
**Labels:** `catalog,store,status`

**Historia de usuario:** Como administrador, quiero cambiar el estado online de un producto para controlar qué productos aparecen en la tienda sin borrarlos del sistema.

**Criterios de aceptación:**
- El producto puede pasar por DRAFT, PUBLISHED, PAUSED y HIDDEN.
- La tienda solo muestra PUBLISHED.
- El endpoint PATCH /api/admin/products/{id}/status actualiza el estado.
- La UI muestra claramente el estado actual.

**Subtareas técnicas sugeridas:**
- [S1-US09-T01] Crear OnlineStatus enum y transición controlada en ProductService.
- [S1-US09-T02] Implementar PATCH /api/admin/products/{id}/status.
- [S1-US09-T03] Filtrar GET /api/store/products por PUBLISHED.
- [S1-US09-T04] Agregar selector de estado en el ABM de productos.
- [S1-US09-T05] Agregar tests para evitar estados inválidos y verificar filtro público.

## S1-US10 — Mostrar catálogo público inicial

**Épica:** EP-03  
**Sprint:** Sprint 1  
**Prioridad:** Alta  
**Story points:** 8  
**Dependencias:** S1-US09  
**Labels:** `store,catalog,frontend`

**Historia de usuario:** Como cliente, quiero ver productos publicados con búsqueda y filtro por categoría para conocer qué vende la dietética antes de comprar.

**Criterios de aceptación:**
- GET /api/store/products soporta q, categoryId, branchId, page y size.
- GET /api/store/products/{id} devuelve detalle de producto.
- Solo se muestran productos PUBLISHED y activos.
- La pantalla pública permite buscar y filtrar sin login.

**Subtareas técnicas sugeridas:**
- [S1-US10-T01] Implementar StoreProductController con listado y detalle.
- [S1-US10-T02] Crear queries paginadas por nombre, categoría y estado publicado.
- [S1-US10-T03] Crear componentes Angular: StoreLayout, ProductGrid, ProductCard, ProductDetail.
- [S1-US10-T04] Implementar StoreProductService en Angular.
- [S1-US10-T05] Agregar estados visuales de loading, empty y error.
- [S1-US10-T06] Agregar tests de endpoint público y filtro por published.

## S1-US11 — Estandarizar errores y validaciones de API

**Épica:** EP-00  
**Sprint:** Sprint 1  
**Prioridad:** Alta  
**Story points:** 5  
**Dependencias:** S1-US04  
**Labels:** `api,error-handling,frontend`

**Historia de usuario:** Como desarrollador, quiero tener respuestas de error uniformes para simplificar el manejo global de errores en frontend y testing.

**Criterios de aceptación:**
- Todas las excepciones de dominio devuelven ApiError con status, code, message, details, timestamp y path.
- Los errores de validación devuelven VALIDATION_ERROR con campos inválidos.
- El frontend tiene interceptor global para mostrar mensajes amigables.
- No se filtran stacktraces al cliente.

**Subtareas técnicas sugeridas:**
- [S1-US11-T01] Crear ApiError record/clase.
- [S1-US11-T02] Crear DomainException base con code y status.
- [S1-US11-T03] Implementar @ControllerAdvice global.
- [S1-US11-T04] Mapear MethodArgumentNotValidException a VALIDATION_ERROR.
- [S1-US11-T05] Crear HttpErrorInterceptor en Angular.
- [S1-US11-T06] Agregar tests de formato de error.

## S1-US12 — Dejar base de testing y datos demo

**Épica:** EP-00  
**Sprint:** Sprint 1  
**Prioridad:** Media  
**Story points:** 5  
**Dependencias:** S1-US03  
**Labels:** `testing,seed,demo`

**Historia de usuario:** Como desarrollador, quiero contar con pruebas y datos mínimos desde el inicio para reducir regresiones en los flujos críticos posteriores.

**Criterios de aceptación:**
- Existe configuración de tests unitarios e integración con PostgreSQL de test.
- Existe seed demo con sucursal, usuarios, categorías y productos.
- Los comandos de test están documentados.
- La demo inicial permite login y navegación básica del catálogo.

**Subtareas técnicas sugeridas:**
- [S1-US12-T01] Configurar JUnit, Mockito y Testcontainers PostgreSQL.
- [S1-US12-T02] Crear clase base para integration tests.
- [S1-US12-T03] Agregar script de seed demo con 15-20 productos representativos.
- [S1-US12-T04] Crear usuario admin, empleado y customer demo.
- [S1-US12-T05] Agregar npm scripts de lint/test/build en frontend.
- [S1-US12-T06] Documentar credenciales demo no productivas.

# Sprint 2 — 2 semanas

**Objetivo del sprint:** Incorporar inventario real por lotes, FEFO, órdenes online pendientes de pago, carrito local, proveedores y consulta de pedidos del cliente.

**Entregables esperados:**
- Stock por lotes como fuente de verdad.
- Ingresos, ajustes y movimientos de stock trazables.
- Política FEFO testeada.
- Creación de orden online PENDING_PAYMENT sin descontar stock.
- Carrito local, checkout inicial y pedidos propios.
- Proveedores y costos manuales.

| Story | Épica | Título | Prioridad | Puntos | Dependencias |
|---|---|---|---|---:|---|
| S2-US01 | EP-05 | Crear modelo de stock por lotes | Alta | 8 | S1-US08 |
| S2-US02 | EP-05 | Registrar ingresos de stock | Alta | 5 | S2-US01 |
| S2-US03 | EP-05 | Implementar política FEFO testeable | Alta | 8 | S2-US01 |
| S2-US04 | EP-05 | Registrar ajustes manuales y consumos internos | Alta | 5 | S2-US01 |
| S2-US05 | EP-03 | Mostrar disponibilidad real en tienda | Alta | 5 | S2-US01 |
| S2-US06 | EP-06 | Crear modelo de órdenes unificadas | Alta | 8 | S2-US01 |
| S2-US07 | EP-09 | Crear base de pagos unificados | Alta | 5 | S2-US06 |
| S2-US08 | EP-06 | Crear orden online pendiente de pago | Alta | 8 | S2-US05,S2-US06,S2-US07 |
| S2-US09 | EP-03 | Implementar carrito local en Angular | Alta | 5 | S2-US05 |
| S2-US10 | EP-03 | Completar flujo frontend de confirmación online | Alta | 5 | S2-US08,S2-US09 |
| S2-US11 | EP-10 | Gestionar proveedores y costos manuales | Media | 8 | S1-US08 |
| S2-US12 | EP-03 | Consultar pedidos propios del cliente | Media | 5 | S2-US08 |

## S2-US01 — Crear modelo de stock por lotes

**Épica:** EP-05  
**Sprint:** Sprint 2  
**Prioridad:** Alta  
**Story points:** 8  
**Dependencias:** S1-US08  
**Labels:** `inventory,stock,db`

**Historia de usuario:** Como administrador, quiero registrar stock por lote, sucursal y vencimiento para tener trazabilidad real y soporte FEFO.

**Criterios de aceptación:**
- Existen tablas stock_lots y stock_movements con migraciones Flyway.
- El stock disponible se calcula como SUM(quantity_available) por producto y sucursal.
- quantity_available soporta decimales numeric(12,3).
- No existe branch_product_stock ni stock_reservations.

**Subtareas técnicas sugeridas:**
- [S2-US01-T01] Crear V4__inventory.sql con stock_lots y stock_movements.
- [S2-US01-T02] Crear StockLot, StockMovement y enums de movimientos.
- [S2-US01-T03] Crear índices por product_id, branch_id y expiration_date.
- [S2-US01-T04] Implementar StockLotRepository con consultas de disponibilidad.
- [S2-US01-T05] Implementar endpoint GET /api/admin/stock/lots.
- [S2-US01-T06] Agregar tests de cálculo de stock disponible.

## S2-US02 — Registrar ingresos de stock

**Épica:** EP-05  
**Sprint:** Sprint 2  
**Prioridad:** Alta  
**Story points:** 5  
**Dependencias:** S2-US01  
**Labels:** `inventory,stock-entry`

**Historia de usuario:** Como administrador, quiero cargar nuevos lotes de productos por sucursal para actualizar disponibilidad real del negocio.

**Criterios de aceptación:**
- POST /api/admin/stock/lots crea lote con producto, sucursal, cantidad, lote, vencimiento y costo opcional.
- Cada ingreso genera stock_movement PURCHASE_ENTRY.
- No se aceptan cantidades negativas o cero.
- La UI permite cargar vencimiento opcional.

**Subtareas técnicas sugeridas:**
- [S2-US02-T01] Crear CreateStockLotRequest y StockLotDto.
- [S2-US02-T02] Implementar InventoryService.createStockLot().
- [S2-US02-T03] Registrar movimiento PURCHASE_ENTRY asociado al lote.
- [S2-US02-T04] Crear pantalla Angular de ingreso de stock.
- [S2-US02-T05] Mostrar stock total actual luego de cargar el lote.
- [S2-US02-T06] Agregar tests de ingreso válido e inválido.

## S2-US03 — Implementar política FEFO testeable

**Épica:** EP-05  
**Sprint:** Sprint 2  
**Prioridad:** Alta  
**Story points:** 8  
**Dependencias:** S2-US01  
**Labels:** `inventory,fefo,unit-tests`

**Historia de usuario:** Como desarrollador, quiero descontar stock priorizando lotes que vencen primero para cumplir la regla de negocio central de inventario.

**Criterios de aceptación:**
- La política ordena por expiration_date ascendente con NULLS LAST.
- Puede descontar de múltiples lotes para cubrir una cantidad.
- Falla con INSUFFICIENT_STOCK si no alcanza el total.
- La lógica pura puede probarse sin levantar Spring.

**Subtareas técnicas sugeridas:**
- [S2-US03-T01] Crear FefoStockDeductionPolicy.
- [S2-US03-T02] Definir estructura DeductionPlan con lotId y quantityToDeduct.
- [S2-US03-T03] Implementar cálculo sobre lista de lotes disponibles.
- [S2-US03-T04] Cubrir casos: un lote, varios lotes, lote sin vencimiento, stock insuficiente y cantidad decimal.
- [S2-US03-T05] Documentar criterio FEFO dentro del módulo inventory.

## S2-US04 — Registrar ajustes manuales y consumos internos

**Épica:** EP-05  
**Sprint:** Sprint 2  
**Prioridad:** Alta  
**Story points:** 5  
**Dependencias:** S2-US01  
**Labels:** `inventory,adjustments,audit`

**Historia de usuario:** Como administrador, quiero ajustar stock con motivo obligatorio para corregir diferencias manteniendo trazabilidad.

**Criterios de aceptación:**
- POST /api/admin/stock/adjustments permite ajuste positivo o negativo con reason obligatorio.
- No se permite dejar un lote con stock negativo.
- Los movimientos quedan registrados como MANUAL_ADJUSTMENT o INTERNAL_CONSUMPTION cuando corresponda.
- EMPLOYEE puede consultar stock pero no realizar ajustes administrativos si se decide restringir por rol.

**Subtareas técnicas sugeridas:**
- [S2-US04-T01] Crear StockAdjustmentRequest.
- [S2-US04-T02] Implementar validación de motivo obligatorio.
- [S2-US04-T03] Implementar ajuste sobre lote específico o producto/sucursal con FEFO para negativos.
- [S2-US04-T04] Registrar StockMovement con created_by_user_id.
- [S2-US04-T05] Crear listado de movimientos filtrable.
- [S2-US04-T06] Agregar tests de ajuste negativo sin stock y reason vacío.

## S2-US05 — Mostrar disponibilidad real en tienda

**Épica:** EP-03  
**Sprint:** Sprint 2  
**Prioridad:** Alta  
**Story points:** 5  
**Dependencias:** S2-US01  
**Labels:** `store,inventory,availability`

**Historia de usuario:** Como cliente, quiero ver stock disponible por sucursal en catálogo y detalle para evitar agregar productos agotados al carrito.

**Criterios de aceptación:**
- La tienda consulta disponibilidad desde stock_lots por branchId.
- El detalle de producto muestra disponible/agregado según stock.
- Si no hay stock, el botón de agregar al carrito queda deshabilitado.
- El backend no expone costos ni datos internos.

**Subtareas técnicas sugeridas:**
- [S2-US05-T01] Extender ProductSummaryDto y ProductDetailDto con availableStock.
- [S2-US05-T02] Optimizar consulta de disponibilidad para listado paginado.
- [S2-US05-T03] Actualizar ProductCard y ProductDetail con stock visible.
- [S2-US05-T04] Validar branchId requerido o usar sucursal default del MVP.
- [S2-US05-T05] Agregar tests de producto publicado sin stock.

## S2-US06 — Crear modelo de órdenes unificadas

**Épica:** EP-06  
**Sprint:** Sprint 2  
**Prioridad:** Alta  
**Story points:** 8  
**Dependencias:** S2-US01  
**Labels:** `orders,db,domain`

**Historia de usuario:** Como desarrollador, quiero representar ventas POS y pedidos online en una misma entidad para evitar duplicación de lógica comercial.

**Criterios de aceptación:**
- Existen tablas orders y order_items con type POS/ONLINE.
- OrderItem guarda snapshot de nombre, barcode, precio y costo cuando exista.
- ONLINE requiere customer_user_id y POS requiere created_by_user_id.
- fulfillment_type queda en PICKUP para el MVP.

**Subtareas técnicas sugeridas:**
- [S2-US06-T01] Crear V5__orders.sql con orders y order_items.
- [S2-US06-T02] Crear entidades Order y OrderItem.
- [S2-US06-T03] Crear enums OrderType, OrderStatus y FulfillmentType.
- [S2-US06-T04] Implementar OrderNumberGenerator.
- [S2-US06-T05] Crear repositorios y DTOs OrderSummaryDto/OrderDetailDto.
- [S2-US06-T06] Agregar constraints y tests de reglas ONLINE/POS.

## S2-US07 — Crear base de pagos unificados

**Épica:** EP-09  
**Sprint:** Sprint 2  
**Prioridad:** Alta  
**Story points:** 5  
**Dependencias:** S2-US06  
**Labels:** `payments,db,domain`

**Historia de usuario:** Como desarrollador, quiero registrar pagos online y presenciales en una tabla común para tener trazabilidad y reportes consistentes.

**Criterios de aceptación:**
- Existe tabla payments con provider, method, status, amount y referencias externas.
- El payment online puede tener cash_session_id null.
- El payment presencial queda preparado para asociarse a cash_session_id en Sprint 3.
- No se almacena información sensible de tarjetas.

**Subtareas técnicas sugeridas:**
- [S2-US07-T01] Crear V6__payments.sql.
- [S2-US07-T02] Crear Payment entity y enums PaymentProvider, PaymentMethod, PaymentStatus.
- [S2-US07-T03] Crear PaymentRepository y PaymentService base.
- [S2-US07-T04] Agregar relación Order -> Payments.
- [S2-US07-T05] Implementar consulta de payments por order.
- [S2-US07-T06] Agregar tests de constraints de método, proveedor y monto.

## S2-US08 — Crear orden online pendiente de pago

**Épica:** EP-06  
**Sprint:** Sprint 2  
**Prioridad:** Alta  
**Story points:** 8  
**Dependencias:** S2-US05,S2-US06,S2-US07  
**Labels:** `orders,customer,checkout`

**Historia de usuario:** Como cliente, quiero confirmar mi carrito y crear una orden online para iniciar el checkout sin descontar stock todavía.

**Criterios de aceptación:**
- POST /api/customer/orders requiere rol CUSTOMER.
- Valida que haya stock suficiente al momento de crear la orden.
- Crea order ONLINE con status PENDING_PAYMENT.
- Crea payment MERCADO_PAGO/CHECKOUT_PRO con status PENDING.
- No descuenta stock en esta etapa.

**Subtareas técnicas sugeridas:**
- [S2-US08-T01] Crear CustomerOrderController.
- [S2-US08-T02] Implementar CreateOnlineOrderRequest con items productId/quantity.
- [S2-US08-T03] Validar productos activos y publicados.
- [S2-US08-T04] Validar stock disponible por sucursal sin modificar lotes.
- [S2-US08-T05] Crear snapshots de cliente y producto en la orden.
- [S2-US08-T06] Calcular subtotal, descuentos en 0 y total.
- [S2-US08-T07] Agregar tests de stock insuficiente, producto inexistente y orden exitosa.

## S2-US09 — Implementar carrito local en Angular

**Épica:** EP-03  
**Sprint:** Sprint 2  
**Prioridad:** Alta  
**Story points:** 5  
**Dependencias:** S2-US05  
**Labels:** `frontend,cart,store`

**Historia de usuario:** Como cliente, quiero agregar productos al carrito local para preparar una compra sin persistir carrito en base.

**Criterios de aceptación:**
- El carrito se guarda en localStorage.
- Permite agregar, quitar, modificar cantidades y vaciar.
- Revalida stock antes de confirmar compra.
- El carrito no queda asociado a usuario en base de datos.

**Subtareas técnicas sugeridas:**
- [S2-US09-T01] Crear CartService con Angular Signals.
- [S2-US09-T02] Crear modelo CartItem con productId, name, price, imageUrl y quantity.
- [S2-US09-T03] Crear componentes CartDrawer/CartPage.
- [S2-US09-T04] Persistir y recuperar carrito desde localStorage.
- [S2-US09-T05] Validar cantidad mayor a cero y no superior al disponible conocido.
- [S2-US09-T06] Agregar tests unitarios del servicio de carrito.

## S2-US10 — Completar flujo frontend de confirmación online

**Épica:** EP-03  
**Sprint:** Sprint 2  
**Prioridad:** Alta  
**Story points:** 5  
**Dependencias:** S2-US08,S2-US09  
**Labels:** `frontend,checkout,customer`

**Historia de usuario:** Como cliente, quiero confirmar mi carrito autenticado para generar una orden pendiente de pago.

**Criterios de aceptación:**
- Si el usuario no está autenticado, se lo redirige a login antes de confirmar.
- El checkout crea una orden ONLINE PENDING_PAYMENT desde el carrito local.
- Al crear la orden se muestra número de orden y estado.
- El carrito no se vacía hasta que el pago sea iniciado/confirmado según decisión UX.

**Subtareas técnicas sugeridas:**
- [S2-US10-T01] Crear CheckoutPage protegida por AuthGuard CUSTOMER.
- [S2-US10-T02] Integrar CartService con CustomerOrderService.
- [S2-US10-T03] Mostrar resumen de compra, cantidades y total.
- [S2-US10-T04] Manejar errores INSUFFICIENT_STOCK y PRODUCT_NOT_FOUND.
- [S2-US10-T05] Crear pantalla de orden creada pendiente de pago.
- [S2-US10-T06] Agregar tests de componente/servicio para confirmación.

## S2-US11 — Gestionar proveedores y costos manuales

**Épica:** EP-10  
**Sprint:** Sprint 2  
**Prioridad:** Media  
**Story points:** 8  
**Dependencias:** S1-US08  
**Labels:** `suppliers,costs,admin`

**Historia de usuario:** Como administrador, quiero registrar proveedores y costos por producto para tener referencia de reposición y margen sin importar listas externas todavía.

**Criterios de aceptación:**
- ADMIN puede crear y editar proveedores.
- ADMIN puede asociar producto-proveedor con costo actual y SKU proveedor.
- El costo no modifica automáticamente el precio de venta.
- La importación automática queda fuera del MVP.

**Subtareas técnicas sugeridas:**
- [S2-US11-T01] Crear V3__suppliers.sql con suppliers y supplier_products.
- [S2-US11-T02] Crear entidades Supplier y SupplierProduct.
- [S2-US11-T03] Implementar endpoints /api/admin/suppliers.
- [S2-US11-T04] Implementar endpoints /api/admin/supplier-products.
- [S2-US11-T05] Crear pantallas Angular de proveedores y asociación con producto.
- [S2-US11-T06] Agregar tests de CUIT duplicado, costo negativo y asociación.

## S2-US12 — Consultar pedidos propios del cliente

**Épica:** EP-03  
**Sprint:** Sprint 2  
**Prioridad:** Media  
**Story points:** 5  
**Dependencias:** S2-US08  
**Labels:** `customer,orders,frontend`

**Historia de usuario:** Como cliente, quiero ver mis pedidos y su estado para saber cuándo puedo retirar la compra.

**Criterios de aceptación:**
- GET /api/customer/orders lista solo órdenes del CUSTOMER autenticado.
- GET /api/customer/orders/{id} impide ver pedidos ajenos.
- El detalle incluye items, total, estado de orden y estado del payment.
- La UI muestra historial de pedidos del cliente.

**Subtareas técnicas sugeridas:**
- [S2-US12-T01] Implementar endpoints de listado y detalle customer.
- [S2-US12-T02] Agregar validación de ownership por customer_user_id.
- [S2-US12-T03] Crear CustomerOrdersPage y CustomerOrderDetailPage.
- [S2-US12-T04] Mostrar estados PENDING_PAYMENT, PAID, PREPARING, READY, DELIVERED, CANCELLED, PAYMENT_FAILED y STOCK_CONFLICT.
- [S2-US12-T05] Agregar tests de acceso prohibido a pedido ajeno.

# Sprint 3 — 2 semanas

**Objetivo del sprint:** Completar los flujos transaccionales principales: Mercado Pago, webhook, descuento FEFO online, caja operativa y venta presencial POS.

**Entregables esperados:**
- Checkout Pro con preferencia de pago.
- Webhook con idempotencia.
- Descuento FEFO al aprobar pago online.
- Caja: apertura, movimientos y cierre con arqueo de efectivo.
- POS transaccional con múltiples medios de pago.
- Tests críticos de pagos/caja/stock.

| Story | Épica | Título | Prioridad | Puntos | Dependencias |
|---|---|---|---|---:|---|
| S3-US01 | EP-04 | Crear adaptador de Mercado Pago | Alta | 5 | S2-US07 |
| S3-US02 | EP-04 | Crear preferencia de pago Checkout Pro | Alta | 5 | S3-US01,S2-US08 |
| S3-US03 | EP-04 | Procesar webhook de Mercado Pago con idempotencia | Alta | 8 | S3-US02 |
| S3-US04 | EP-05 | Descontar stock FEFO al aprobar pago online | Alta | 8 | S3-US03,S2-US03 |
| S3-US05 | EP-03 | Integrar checkout frontend con Mercado Pago | Alta | 5 | S3-US02 |
| S3-US06 | EP-07 | Abrir caja operativa por sucursal | Alta | 8 | S2-US07 |
| S3-US07 | EP-07 | Registrar movimientos manuales de caja | Media | 5 | S3-US06 |
| S3-US08 | EP-07 | Cerrar caja con arqueo de efectivo | Alta | 5 | S3-US06,S3-US07 |
| S3-US09 | EP-08 | Buscar productos en POS por nombre o código de barras | Media | 5 | S1-US08,S2-US05 |
| S3-US10 | EP-08 | Crear venta presencial transaccional | Alta | 8 | S3-US06,S3-US09,S2-US03 |
| S3-US11 | EP-08 | Construir pantalla POS completa | Alta | 8 | S3-US10 |
| S3-US12 | EP-00 | Probar flujos críticos de pagos, caja y POS | Alta | 5 | S3-US03,S3-US04,S3-US08,S3-US10 |

## S3-US01 — Crear adaptador de Mercado Pago

**Épica:** EP-04  
**Sprint:** Sprint 3  
**Prioridad:** Alta  
**Story points:** 5  
**Dependencias:** S2-US07  
**Labels:** `mercadopago,payments,adapter`

**Historia de usuario:** Como desarrollador, quiero encapsular la integración con Mercado Pago detrás de una interfaz para poder testear con mock y cambiar implementación sin rediseñar pagos.

**Criterios de aceptación:**
- Existe PaymentGateway con createPreference, checkPayment y verifyWebhookSignature.
- Existe MercadoPagoGateway como implementación real/configurable.
- Los tests pueden usar FakePaymentGateway.
- Las credenciales se leen desde variables de entorno.

**Subtareas técnicas sugeridas:**
- [S3-US01-T01] Crear interfaz PaymentGateway.
- [S3-US01-T02] Crear DTO interno PaymentPreferenceResult con initPoint y preferenceId.
- [S3-US01-T03] Crear MercadoPagoGateway con configuración externa.
- [S3-US01-T04] Crear FakePaymentGateway para tests/dev.
- [S3-US01-T05] Agregar properties MP_ACCESS_TOKEN, MP_WEBHOOK_SECRET, MP_SUCCESS_URL, MP_FAILURE_URL.
- [S3-US01-T06] Documentar cómo usar sandbox/mock.

## S3-US02 — Crear preferencia de pago Checkout Pro

**Épica:** EP-04  
**Sprint:** Sprint 3  
**Prioridad:** Alta  
**Story points:** 5  
**Dependencias:** S3-US01,S2-US08  
**Labels:** `mercadopago,checkout,customer`

**Historia de usuario:** Como cliente, quiero iniciar el pago de mi orden en Mercado Pago para completar la compra en una pasarela segura.

**Criterios de aceptación:**
- POST /api/customer/orders/{orderId}/checkout/mp crea o reutiliza preferencia.
- El endpoint es idempotente si la orden ya tiene provider_preference_id.
- Solo se permite para órdenes propias en PENDING_PAYMENT.
- Devuelve initPoint y preferenceId.

**Subtareas técnicas sugeridas:**
- [S3-US02-T01] Crear MercadoPagoService.createCheckoutPreference().
- [S3-US02-T02] Validar estado de orden PENDING_PAYMENT.
- [S3-US02-T03] Validar ownership CUSTOMER.
- [S3-US02-T04] Guardar provider_preference_id y external_reference en payment.
- [S3-US02-T05] Mapear datos de order/items al request de MP.
- [S3-US02-T06] Agregar tests de preferencia nueva, preferencia existente y orden inválida.

## S3-US03 — Procesar webhook de Mercado Pago con idempotencia

**Épica:** EP-04  
**Sprint:** Sprint 3  
**Prioridad:** Alta  
**Story points:** 8  
**Dependencias:** S3-US02  
**Labels:** `webhook,mercadopago,idempotency`

**Historia de usuario:** Como sistema, quiero recibir confirmaciones de pago desde Mercado Pago para actualizar orden y pago sin duplicar operaciones.

**Criterios de aceptación:**
- POST /api/webhooks/mercadopago verifica firma antes de procesar.
- Consulta el estado real del pago en el proveedor/gateway.
- Si el mismo provider_payment_id ya fue procesado, responde OK sin duplicar stock_movements.
- REJECTED actualiza payment y order a PAYMENT_FAILED.

**Subtareas técnicas sugeridas:**
- [S3-US03-T01] Crear endpoint público MercadoPagoWebhookController.
- [S3-US03-T02] Implementar verificación de firma en gateway.
- [S3-US03-T03] Implementar búsqueda por provider_payment_id/provider_preference_id/external_reference.
- [S3-US03-T04] Agregar control idempotente antes de modificar orden o stock.
- [S3-US03-T05] Mapear estados MP a PaymentStatus interno.
- [S3-US03-T06] Registrar metadata JSONB del webhook sin datos sensibles.
- [S3-US03-T07] Agregar integration tests de webhook duplicado y rechazado.

## S3-US04 — Descontar stock FEFO al aprobar pago online

**Épica:** EP-05  
**Sprint:** Sprint 3  
**Prioridad:** Alta  
**Story points:** 8  
**Dependencias:** S3-US03,S2-US03  
**Labels:** `inventory,fefo,online-payments`

**Historia de usuario:** Como sistema, quiero descontar stock cuando Mercado Pago aprueba la orden para mantener sincronizado stock real y tienda online.

**Criterios de aceptación:**
- Al aprobar pago online, se bloquean lotes con SELECT FOR UPDATE.
- Se descuenta stock FEFO por cada item.
- Se crean movimientos ONLINE_ORDER con order_id.
- La order pasa a PAID y payment a APPROVED.
- Si no hay stock suficiente, la order pasa a STOCK_CONFLICT y no queda stock negativo.

**Subtareas técnicas sugeridas:**
- [S3-US04-T01] Implementar InventoryService.deductForOnlineOrder() transaccional.
- [S3-US04-T02] Crear consulta repository con bloqueo pesimista ordenada por vencimiento.
- [S3-US04-T03] Registrar StockMovement por lote afectado.
- [S3-US04-T04] Actualizar paid_at y approved_at.
- [S3-US04-T05] Implementar manejo STOCK_CONFLICT.
- [S3-US04-T06] Agregar tests concurrentes básicos de no sobreventa.
- [S3-US04-T07] Agregar tests de descuento de varios lotes.

## S3-US05 — Integrar checkout frontend con Mercado Pago

**Épica:** EP-03  
**Sprint:** Sprint 3  
**Prioridad:** Alta  
**Story points:** 5  
**Dependencias:** S3-US02  
**Labels:** `frontend,mercadopago,checkout`

**Historia de usuario:** Como cliente, quiero ser redirigido a Mercado Pago y volver a ver el resultado para completar la compra online de punta a punta.

**Criterios de aceptación:**
- La UI llama al endpoint de checkout y redirige a initPoint.
- Existe pantalla de resultado de pago que consulta el estado real de la orden.
- El cliente ve mensajes claros para pago pendiente, aprobado, rechazado o conflicto de stock.
- No se muestra información interna de pagos.

**Subtareas técnicas sugeridas:**
- [S3-US05-T01] Crear CustomerCheckoutService.
- [S3-US05-T02] Agregar botón Pagar con Mercado Pago en orden pendiente.
- [S3-US05-T03] Implementar window.location.href hacia initPoint.
- [S3-US05-T04] Crear PaymentResultPage para success/failure/pending.
- [S3-US05-T05] Consultar GET /api/customer/orders/{id} al volver.
- [S3-US05-T06] Manejar estado asincrónico donde el webhook aún no impactó.

## S3-US06 — Abrir caja operativa por sucursal

**Épica:** EP-07  
**Sprint:** Sprint 3  
**Prioridad:** Alta  
**Story points:** 8  
**Dependencias:** S2-US07  
**Labels:** `cash,backoffice,pos`

**Historia de usuario:** Como empleado, quiero abrir caja con monto inicial de efectivo para comenzar ventas presenciales controlando efectivo.

**Criterios de aceptación:**
- ADMIN/MANAGER/EMPLOYEE pueden abrir caja.
- Solo puede existir una caja OPEN por sucursal.
- La caja guarda opened_by_user_id, branch_id, opening_cash_amount y opening_notes.
- GET /api/admin/cash-sessions/current devuelve la caja abierta actual.

**Subtareas técnicas sugeridas:**
- [S3-US06-T01] Crear V7__cash.sql con cash_sessions y cash_movements.
- [S3-US06-T02] Crear CashSession entity, repository y status enum.
- [S3-US06-T03] Implementar CashService.openCashSession().
- [S3-US06-T04] Validar una caja abierta por sucursal.
- [S3-US06-T05] Crear endpoints open/current.
- [S3-US06-T06] Crear UI de apertura de caja.
- [S3-US06-T07] Agregar tests de caja duplicada y apertura por empleado.

## S3-US07 — Registrar movimientos manuales de caja

**Épica:** EP-07  
**Sprint:** Sprint 3  
**Prioridad:** Media  
**Story points:** 5  
**Dependencias:** S3-US06  
**Labels:** `cash,movements`

**Historia de usuario:** Como empleado, quiero registrar ingresos, egresos o ajustes manuales para mantener trazabilidad del efectivo y movimientos no asociados a ventas.

**Criterios de aceptación:**
- Solo se pueden registrar movimientos si la caja está OPEN.
- Todo movimiento exige motivo.
- Se registran type CASH_IN/CASH_OUT/ADJUSTMENT y method CASH/TRANSFER/OTHER.
- El movimiento queda asociado al usuario que lo creó.

**Subtareas técnicas sugeridas:**
- [S3-US07-T01] Crear CashMovement entity y enums.
- [S3-US07-T02] Implementar POST /api/admin/cash-sessions/{id}/movements.
- [S3-US07-T03] Validar reason obligatorio y amount distinto de cero.
- [S3-US07-T04] Actualizar detalle de caja con movimientos manuales.
- [S3-US07-T05] Crear formulario Angular de movimiento manual.
- [S3-US07-T06] Agregar tests de caja cerrada, reason vacío y usuario creador.

## S3-US08 — Cerrar caja con arqueo de efectivo

**Épica:** EP-07  
**Sprint:** Sprint 3  
**Prioridad:** Alta  
**Story points:** 5  
**Dependencias:** S3-US06,S3-US07  
**Labels:** `cash,closing,reports`

**Historia de usuario:** Como empleado, quiero cerrar caja informando efectivo contado para comparar efectivo esperado contra físico y justificar diferencias.

**Criterios de aceptación:**
- El cierre calcula expected_cash_amount usando apertura + pagos CASH + movimientos CASH.
- El cierre muestra totales informativos por CASH, QR, TRANSFER, DEBIT_CARD y CREDIT_CARD.
- Si countedCashAmount difiere de expected, exige cashDifferenceReason.
- La diferencia no bloquea el cierre si está justificada.

**Subtareas técnicas sugeridas:**
- [S3-US08-T01] Crear CashCloseCalculator.
- [S3-US08-T02] Implementar POST /api/admin/cash-sessions/{id}/close.
- [S3-US08-T03] Calcular totalsByMethod desde payments asociados a la caja.
- [S3-US08-T04] Persistir counted, expected, difference, reason, closed_by_user_id y closed_at.
- [S3-US08-T05] Crear UI de cierre con resumen de métodos y campo de efectivo contado.
- [S3-US08-T06] Agregar tests de cierre sin diferencia, con diferencia sin razón y con diferencia justificada.

## S3-US09 — Buscar productos en POS por nombre o código de barras

**Épica:** EP-08  
**Sprint:** Sprint 3  
**Prioridad:** Media  
**Story points:** 5  
**Dependencias:** S1-US08,S2-US05  
**Labels:** `pos,barcode,frontend`

**Historia de usuario:** Como empleado, quiero buscar productos rápidamente en la venta presencial para reducir tiempo operativo y soportar lector como input teclado.

**Criterios de aceptación:**
- El POS permite buscar por nombre o barcode.
- La búsqueda muestra precio y stock disponible en la sucursal del empleado.
- El lector de código de barras funciona como entrada de teclado.
- No se pueden agregar productos sin stock.

**Subtareas técnicas sugeridas:**
- [S3-US09-T01] Crear endpoint específico o reutilizar catálogo admin con filtro q/barcode.
- [S3-US09-T02] Optimizar índice de barcode.
- [S3-US09-T03] Crear POSProductSearchComponent.
- [S3-US09-T04] Agregar input enfocado para scanner.
- [S3-US09-T05] Mostrar stock disponible y precio.
- [S3-US09-T06] Agregar tests de búsqueda por barcode.

## S3-US10 — Crear venta presencial transaccional

**Épica:** EP-08  
**Sprint:** Sprint 3  
**Prioridad:** Alta  
**Story points:** 8  
**Dependencias:** S3-US06,S3-US09,S2-US03  
**Labels:** `pos,transactions,inventory,payments`

**Historia de usuario:** Como empleado, quiero cobrar una venta presencial con distintos medios de pago para registrar venta, pago, caja y descuento de stock en una operación segura.

**Criterios de aceptación:**
- POST /api/admin/pos/sales exige caja abierta.
- Acepta métodos CASH, QR, TRANSFER, DEBIT_CARD y CREDIT_CARD.
- Crea order POS, order_items, payment APPROVED con cash_session_id y stock_movements POS_SALE.
- Descuenta stock FEFO dentro de la misma transacción.
- Si no hay stock o caja abierta, no se crea venta parcial.

**Subtareas técnicas sugeridas:**
- [S3-US10-T01] Implementar PosSaleService.createSale() con @Transactional.
- [S3-US10-T02] Validar caja OPEN de la sucursal del usuario.
- [S3-US10-T03] Bloquear lotes con FOR UPDATE y aplicar FEFO.
- [S3-US10-T04] Crear order POS con status PAID o COMPLETED según enum final adoptado.
- [S3-US10-T05] Crear payment provider MANUAL/BANK/CARD_TERMINAL según método.
- [S3-US10-T06] Crear snapshots de items.
- [S3-US10-T07] Registrar movimientos POS_SALE por lote.
- [S3-US10-T08] Agregar integration tests de venta cash, QR, stock insuficiente y sin caja.

## S3-US11 — Construir pantalla POS completa

**Épica:** EP-08  
**Sprint:** Sprint 3  
**Prioridad:** Alta  
**Story points:** 8  
**Dependencias:** S3-US10  
**Labels:** `pos,frontend,ux`

**Historia de usuario:** Como empleado, quiero armar una venta, modificar cantidades y cobrar para operar ventas presenciales desde una interfaz rápida.

**Criterios de aceptación:**
- La pantalla POS permite agregar productos, modificar cantidades, quitar items y ver total.
- Antes de cobrar verifica caja abierta actual.
- Permite seleccionar método de pago.
- Muestra comprobante/resumen simple al finalizar.

**Subtareas técnicas sugeridas:**
- [S3-US11-T01] Crear AdminPosPage.
- [S3-US11-T02] Integrar POSProductSearchComponent con carrito POS en memoria.
- [S3-US11-T03] Implementar selector de método de pago.
- [S3-US11-T04] Mostrar total y validaciones de stock/cantidad.
- [S3-US11-T05] Consumir POST /api/admin/pos/sales.
- [S3-US11-T06] Mostrar resultado con número de orden y método de pago.
- [S3-US11-T07] Agregar manejo de errores CASH_SESSION_REQUIRED e INSUFFICIENT_STOCK.

## S3-US12 — Probar flujos críticos de pagos, caja y POS

**Épica:** EP-00  
**Sprint:** Sprint 3  
**Prioridad:** Alta  
**Story points:** 5  
**Dependencias:** S3-US03,S3-US04,S3-US08,S3-US10  
**Labels:** `testing,critical-flows`

**Historia de usuario:** Como desarrollador, quiero tener cobertura de los flujos transaccionales principales para evitar regresiones en las partes más riesgosas del MVP.

**Criterios de aceptación:**
- Hay integration tests para webhook aprobado, webhook duplicado y stock conflict.
- Hay integration tests para abrir/cerrar caja y venta POS.
- Hay unit tests para CashCloseCalculator y FefoStockDeductionPolicy.
- Las pruebas corren en el pipeline local documentado.

**Subtareas técnicas sugeridas:**
- [S3-US12-T01] Crear fixtures de productos, lotes, usuarios y caja.
- [S3-US12-T02] Testear webhook aprobado con descuento FEFO.
- [S3-US12-T03] Testear webhook duplicado sin doble descuento.
- [S3-US12-T04] Testear POS con caja abierta y sin caja.
- [S3-US12-T05] Testear cierre de caja con diferencia y razón obligatoria.
- [S3-US12-T06] Agregar comando único backend test.

# Sprint 4 — 2 semanas

**Objetivo del sprint:** Cerrar operación backoffice y calidad final: estados de pedidos, cancelaciones, reportes, recomendaciones, auditoría, seguridad, E2E, demo y documentación.

**Entregables esperados:**
- Preparación, listo para retiro, entrega y cancelación con reversa de stock.
- Backoffice de pedidos.
- Dashboard, reporte de caja y recomendaciones.
- Auditoría de acciones críticas.
- Hardening de seguridad, UX responsive, E2E y despliegue demo.
- Documentación final y tablero Jira listo para tutor.

| Story | Épica | Título | Prioridad | Puntos | Dependencias |
|---|---|---|---|---:|---|
| S4-US01 | EP-06 | Gestionar preparación y retiro de pedidos online | Alta | 8 | S3-US04,S2-US12 |
| S4-US02 | EP-06 | Cancelar orden y revertir stock | Alta | 8 | S3-US04,S4-US01 |
| S4-US03 | EP-06 | Administrar pedidos desde backoffice | Alta | 5 | S4-US01,S4-US02 |
| S4-US04 | EP-11 | Crear dashboard operativo del día | Media | 8 | S3-US10,S4-US03 |
| S4-US05 | EP-11 | Generar reporte de cierre de caja | Media | 5 | S3-US08,S3-US10 |
| S4-US06 | EP-11 | Implementar recomendaciones por reglas | Media | 8 | S2-US01,S4-US04 |
| S4-US07 | EP-00 | Auditar acciones críticas | Alta | 5 | S3-US08,S4-US02 |
| S4-US08 | EP-00 | Pulir seguridad, permisos y guards | Alta | 5 | S4-US03,S4-US07 |
| S4-US09 | EP-00 | Mejorar usabilidad responsive del MVP | Media | 5 | S3-US11,S4-US04 |
| S4-US10 | EP-00 | Cubrir flujos E2E principales | Alta | 8 | S4-US09,S4-US08 |
| S4-US11 | EP-00 | Preparar despliegue de demo | Alta | 5 | S4-US10 |
| S4-US12 | EP-00 | Cerrar documentación académica y evidencia Jira | Media | 5 | S4-US11 |

## S4-US01 — Gestionar preparación y retiro de pedidos online

**Épica:** EP-06  
**Sprint:** Sprint 4  
**Prioridad:** Alta  
**Story points:** 8  
**Dependencias:** S3-US04,S2-US12  
**Labels:** `orders,fulfillment,pickup`

**Historia de usuario:** Como empleado, quiero cambiar estados de pedidos online hasta entregarlos para organizar el retiro en sucursal con trazabilidad.

**Criterios de aceptación:**
- El empleado puede pasar PAID -> PREPARING -> READY -> DELIVERED.
- No se puede preparar una orden sin pago aprobado.
- DELIVERED no descuenta stock nuevamente.
- Se registran timestamps prepared_at y delivered_at.

**Subtareas técnicas sugeridas:**
- [S4-US01-T01] Implementar PATCH /api/admin/orders/{id}/prepare.
- [S4-US01-T02] Implementar PATCH /api/admin/orders/{id}/ready.
- [S4-US01-T03] Implementar PATCH /api/admin/orders/{id}/delivered.
- [S4-US01-T04] Validar transiciones con OrderStatePolicy.
- [S4-US01-T05] Crear UI admin de cambio de estado.
- [S4-US01-T06] Agregar tests de transiciones válidas e inválidas.

## S4-US02 — Cancelar orden y revertir stock

**Épica:** EP-06  
**Sprint:** Sprint 4  
**Prioridad:** Alta  
**Story points:** 8  
**Dependencias:** S3-US04,S4-US01  
**Labels:** `orders,cancellation,inventory`

**Historia de usuario:** Como administrador, quiero cancelar una orden con motivo para corregir pedidos problemáticos sin perder trazabilidad.

**Criterios de aceptación:**
- PATCH /api/admin/orders/{id}/cancel exige reason.
- Si la orden ya descontó stock, se revierte contra los lotes originales.
- Se crean movimientos CANCELLATION_RETURN.
- No se puede cancelar una orden DELIVERED salvo decisión explícita fuera del MVP.

**Subtareas técnicas sugeridas:**
- [S4-US02-T01] Implementar OrderCancellationService transaccional.
- [S4-US02-T02] Buscar stock_movements originales por order_id.
- [S4-US02-T03] Restaurar quantity_available en los mismos stock_lot_id.
- [S4-US02-T04] Crear movimientos inversos CANCELLATION_RETURN.
- [S4-US02-T05] Actualizar status CANCELLED y cancellation_reason.
- [S4-US02-T06] Actualizar/reflejar payment CANCELLED o REFUNDED según escenario MVP.
- [S4-US02-T07] Agregar tests de cancelación PAID, PREPARING, READY y estado inválido.

## S4-US03 — Administrar pedidos desde backoffice

**Épica:** EP-06  
**Sprint:** Sprint 4  
**Prioridad:** Alta  
**Story points:** 5  
**Dependencias:** S4-US01,S4-US02  
**Labels:** `orders,backoffice,frontend`

**Historia de usuario:** Como empleado, quiero listar, filtrar y ver detalle de pedidos online y ventas POS para operar preparación, entrega y control de órdenes.

**Criterios de aceptación:**
- GET /api/admin/orders filtra por status, branchId, type y rango de fechas.
- El detalle muestra items, pagos y datos snapshot del cliente.
- MANAGER/EMPLOYEE ven órdenes de su sucursal; ADMIN puede ver global.
- La UI permite acceder a acciones permitidas según estado.

**Subtareas técnicas sugeridas:**
- [S4-US03-T01] Implementar filtros de OrderRepository.
- [S4-US03-T02] Crear AdminOrdersPage con tabla paginada.
- [S4-US03-T03] Crear AdminOrderDetailPage.
- [S4-US03-T04] Mostrar pagos asociados y estado actual.
- [S4-US03-T05] Mostrar acciones prepare/ready/delivered/cancel según estado.
- [S4-US03-T06] Agregar tests de filtros y permisos por rol/sucursal.

## S4-US04 — Crear dashboard operativo del día

**Épica:** EP-11  
**Sprint:** Sprint 4  
**Prioridad:** Media  
**Story points:** 8  
**Dependencias:** S3-US10,S4-US03  
**Labels:** `reports,dashboard,analytics`

**Historia de usuario:** Como administrador, quiero ver métricas principales del negocio para tomar decisiones rápidas con datos actuales.

**Criterios de aceptación:**
- GET /api/admin/reports/dashboard devuelve ventas del día, online, POS, pendingOrders, lowStockProducts, expiringLots y topProducts.
- ADMIN ve datos globales; MANAGER ve datos de su sucursal.
- El dashboard carga rápido y no bloquea tareas críticas.
- La UI muestra tarjetas claras y tabla de productos top.

**Subtareas técnicas sugeridas:**
- [S4-US04-T01] Crear ReportService.dashboard().
- [S4-US04-T02] Crear queries agregadas para ventas por tipo y día.
- [S4-US04-T03] Crear query de productos más vendidos.
- [S4-US04-T04] Crear query de pedidos pendientes.
- [S4-US04-T05] Crear DashboardPage Angular con cards y tabla.
- [S4-US04-T06] Agregar tests de agregación por sucursal y rol.

## S4-US05 — Generar reporte de cierre de caja

**Épica:** EP-11  
**Sprint:** Sprint 4  
**Prioridad:** Media  
**Story points:** 5  
**Dependencias:** S3-US08,S3-US10  
**Labels:** `reports,cash,backoffice`

**Historia de usuario:** Como administrador, quiero ver detalle de un cierre con totales por medio de pago para auditar caja y entender diferencias de efectivo.

**Criterios de aceptación:**
- GET /api/admin/reports/cash-session/{id} devuelve sesión, pagos, totales por método, efectivo esperado, contado, diferencia y razón.
- El reporte separa arqueo de efectivo de totales informativos no cash.
- La UI permite consultar historial y detalle de cierres.
- Los empleados solo acceden según sucursal/permisos definidos.

**Subtareas técnicas sugeridas:**
- [S4-US05-T01] Implementar CashReportDto.
- [S4-US05-T02] Crear query de totalsByMethod desde payments.
- [S4-US05-T03] Crear pantalla CashSessionHistoryPage.
- [S4-US05-T04] Crear pantalla CashSessionDetailReportPage.
- [S4-US05-T05] Mostrar apertura, cierre, operador y movimientos manuales.
- [S4-US05-T06] Agregar tests de totales por CASH/QR/TRANSFER/TARJETAS.

## S4-US06 — Implementar recomendaciones por reglas

**Épica:** EP-11  
**Sprint:** Sprint 4  
**Prioridad:** Media  
**Story points:** 8  
**Dependencias:** S2-US01,S4-US04  
**Labels:** `recommendations,rules,reports`

**Historia de usuario:** Como administrador, quiero recibir sugerencias de reposición, vencimientos y rotación para detectar problemas operativos sin revisar todo manualmente.

**Criterios de aceptación:**
- GET /api/admin/recommendations devuelve LOW_STOCK, EXPIRING_SOON, HIGH_ROTATION y NO_MOVEMENT cuando hay datos suficientes.
- No recomienda productos sin stock al cliente ni inventa productos.
- No expone costos ni márgenes.
- Cada recomendación incluye mensaje y urgencia.

**Subtareas técnicas sugeridas:**
- [S4-US06-T01] Crear RecommendationService rule-based.
- [S4-US06-T02] Implementar regla LOW_STOCK usando products.minimum_stock y stock disponible.
- [S4-US06-T03] Implementar regla EXPIRING_SOON con lotes próximos a vencer.
- [S4-US06-T04] Implementar regla HIGH_ROTATION con order_items recientes.
- [S4-US06-T05] Implementar regla NO_MOVEMENT con productos sin ventas recientes.
- [S4-US06-T06] Crear panel Angular de recomendaciones.
- [S4-US06-T07] Agregar tests de cada regla con datos controlados.

## S4-US07 — Auditar acciones críticas

**Épica:** EP-00  
**Sprint:** Sprint 4  
**Prioridad:** Alta  
**Story points:** 5  
**Dependencias:** S3-US08,S4-US02  
**Labels:** `audit,security,traceability`

**Historia de usuario:** Como administrador, quiero tener registro de cambios sensibles para poder explicar diferencias, cancelaciones y cambios de datos.

**Criterios de aceptación:**
- Se registran cambios de precio, ajustes de stock, cancelaciones, apertura/cierre de caja, movimientos de caja, confirmación de pago y altas/bajas de usuarios internos.
- Cada audit_log guarda usuario, acción, entidad, entity_id, descripción y fecha.
- No registra datos sensibles de tarjetas o contraseñas.
- Existe consulta administrativa básica de auditoría.

**Subtareas técnicas sugeridas:**
- [S4-US07-T01] Crear V9__audit.sql y AuditLog entity.
- [S4-US07-T02] Crear AuditService.record().
- [S4-US07-T03] Integrar auditoría en ProductService para cambios de precio.
- [S4-US07-T04] Integrar auditoría en InventoryService para ajustes.
- [S4-US07-T05] Integrar auditoría en CashService para apertura/cierre/movimientos.
- [S4-US07-T06] Integrar auditoría en Payment/Webhook para confirmaciones.
- [S4-US07-T07] Crear endpoint GET /api/admin/audit-logs con filtros básicos.
- [S4-US07-T08] Agregar tests de generación de audit logs.

## S4-US08 — Pulir seguridad, permisos y guards

**Épica:** EP-00  
**Sprint:** Sprint 4  
**Prioridad:** Alta  
**Story points:** 5  
**Dependencias:** S4-US03,S4-US07  
**Labels:** `security,rbac,frontend`

**Historia de usuario:** Como desarrollador, quiero cerrar accesos indebidos por rol y sucursal para evitar exposición de módulos internos a usuarios incorrectos.

**Criterios de aceptación:**
- CUSTOMER no accede a /api/admin/** ni rutas backoffice.
- EMPLOYEE no accede a reportes ni ABMs restringidos si no corresponde.
- MANAGER opera solo su sucursal.
- Angular oculta menús según rol además de validar en backend.

**Subtareas técnicas sugeridas:**
- [S4-US08-T01] Revisar @PreAuthorize en todos los controllers.
- [S4-US08-T02] Crear util de seguridad para branch scope.
- [S4-US08-T03] Completar AdminGuard y CustomerGuard en Angular.
- [S4-US08-T04] Implementar menú dinámico por rol.
- [S4-US08-T05] Agregar tests de acceso prohibido por rol.
- [S4-US08-T06] Validar que los endpoints customer no expongan pedidos ajenos.

## S4-US09 — Mejorar usabilidad responsive del MVP

**Épica:** EP-00  
**Sprint:** Sprint 4  
**Prioridad:** Media  
**Story points:** 5  
**Dependencias:** S3-US11,S4-US04  
**Labels:** `frontend,ux,responsive`

**Historia de usuario:** Como usuario, quiero usar las pantallas principales en escritorio y móvil para hacer viable el uso real en comercio y la demo ante tutor.

**Criterios de aceptación:**
- Catálogo, carrito, login, pedidos, POS, caja y dashboard son usables en pantallas comunes.
- Las tablas grandes tienen paginación o diseño compacto.
- Las alertas críticas son visibles en el panel principal.
- Los formularios muestran validaciones claras.

**Subtareas técnicas sugeridas:**
- [S4-US09-T01] Revisar layout público de tienda en mobile.
- [S4-US09-T02] Revisar layout backoffice y navegación lateral/topbar.
- [S4-US09-T03] Agregar estados empty/loading/error consistentes.
- [S4-US09-T04] Optimizar formularios de stock, caja y POS.
- [S4-US09-T05] Agregar confirmaciones para acciones destructivas: cancelar orden, cerrar caja, desactivar producto.
- [S4-US09-T06] Corregir problemas de accesibilidad básicos en labels, botones y foco.

## S4-US10 — Cubrir flujos E2E principales

**Épica:** EP-00  
**Sprint:** Sprint 4  
**Prioridad:** Alta  
**Story points:** 8  
**Dependencias:** S4-US09,S4-US08  
**Labels:** `testing,e2e,demo`

**Historia de usuario:** Como desarrollador, quiero validar los caminos completos del MVP para demostrar que el sistema funciona de punta a punta.

**Criterios de aceptación:**
- Existe prueba E2E o guion automatizado para registro/login/customer checkout mock.
- Existe prueba E2E o guion automatizado para caja + POS + cierre.
- Existe prueba E2E o guion automatizado para preparación y entrega de pedido.
- Los resultados quedan documentados para defensa/demo.

**Subtareas técnicas sugeridas:**
- [S4-US10-T01] Definir herramienta E2E o scripts de prueba manual reproducibles.
- [S4-US10-T02] Crear dataset estable de demo.
- [S4-US10-T03] Probar flujo customer: registro, catálogo, carrito, orden, checkout mock, pedido.
- [S4-US10-T04] Probar flujo empleado: abrir caja, vender POS, cerrar caja.
- [S4-US10-T05] Probar flujo admin: stock, cancelación, reportes y recomendaciones.
- [S4-US10-T06] Documentar bugs encontrados y correcciones aplicadas.

## S4-US11 — Preparar despliegue de demo

**Épica:** EP-00  
**Sprint:** Sprint 4  
**Prioridad:** Alta  
**Story points:** 5  
**Dependencias:** S4-US10  
**Labels:** `deploy,docker,demo`

**Historia de usuario:** Como desarrollador, quiero desplegar una versión demostrable del MVP para mostrar el sistema funcionando fuera del entorno local.

**Criterios de aceptación:**
- Existe configuración de demo con Docker Compose o plataforma elegida.
- Las variables sensibles se gestionan por entorno.
- Uploads de imágenes quedan servidos como estáticos o alternativa definida.
- La demo permite ejecutar los flujos principales con usuarios seed.

**Subtareas técnicas sugeridas:**
- [S4-US11-T01] Crear Dockerfile backend.
- [S4-US11-T02] Crear build de frontend y configuración de servidor estático/proxy.
- [S4-US11-T03] Configurar CORS para ambiente demo.
- [S4-US11-T04] Configurar variables de Mercado Pago sandbox/mock.
- [S4-US11-T05] Crear script de migración/seed para demo.
- [S4-US11-T06] Documentar pasos de despliegue y rollback simple.
- [S4-US11-T07] Validar demo desde navegador limpio.

## S4-US12 — Cerrar documentación académica y evidencia Jira

**Épica:** EP-00  
**Sprint:** Sprint 4  
**Prioridad:** Media  
**Story points:** 5  
**Dependencias:** S4-US11  
**Labels:** `documentation,jira,academic`

**Historia de usuario:** Como desarrollador, quiero dejar documentación final alineada al MVP implementado para facilitar revisión del tutor y defensa del proyecto.

**Criterios de aceptación:**
- La documentación refleja el alcance real implementado.
- Quedan capturas o evidencias de los flujos principales.
- El README explica arquitectura, ejecución, usuarios demo y decisiones.
- El tablero Jira tiene épicas, historias y tareas actualizadas con estados reales.

**Subtareas técnicas sugeridas:**
- [S4-US12-T01] Actualizar README funcional y técnico.
- [S4-US12-T02] Actualizar documentación de endpoints implementados.
- [S4-US12-T03] Agregar diagrama simple de arquitectura final.
- [S4-US12-T04] Preparar guion de demo: customer, empleado, admin.
- [S4-US12-T05] Adjuntar capturas de catálogo, checkout, caja, POS, reportes y recomendaciones.
- [S4-US12-T06] Revisar que Jira tenga dependencias, story points y criterios de aceptación completos.

# Riesgos y controles por sprint

| Riesgo | Control recomendado | Sprint donde mirar |
|---|---|---|
| Scope creep por funcionalidades post-MVP como envío, multiempresa o importación automática | Mantenerlos fuera del sprint backlog salvo que el tutor los exija formalmente. | Todos |
| Sobreventa por concurrencia entre POS y webhook online | Usar transacciones y bloqueo pesimista SELECT FOR UPDATE en stock_lots. | Sprint 2-3 |
| Webhook duplicado de Mercado Pago descontando dos veces | Idempotencia por provider_payment_id / external_reference y tests de duplicado. | Sprint 3 |
| Caja mal interpretada como control de todos los medios de pago | Arqueo solo de efectivo; totales por otros métodos son informativos. | Sprint 3-4 |
| Roles mezclados entre CUSTOMER y usuarios internos | CUSTOMER con branch_id null y rutas customer/admin separadas. | Sprint 1-4 |
| Reportes pesados bloqueando operación diaria | Queries agregadas paginadas y dashboard acotado al MVP. | Sprint 4 |

# Orden recomendado de implementación dentro de cada sprint

1. Primero migraciones/modelo de datos y entities.
2. Después servicios de dominio con tests unitarios.
3. Luego controllers/endpoints e integration tests.
4. Finalmente pantallas Angular, guards, interceptores y validaciones UX.
5. Cerrar cada sprint con demo integrada, no solo endpoints aislados.
