---
title: "Diseno Backend"
tags:
  - arquitectura
  - backend
  - spring-boot
  - arquitectura-detallada
---

# Diseno Backend

> [!info] Arquitectura detallada del backend: organizacion por features, capas pragmaticas, transacciones, seguridad, manejo de errores e integraciones.

---

## Principios arquitectonicos

1. **Monolito modular con organizacion por feature**: cada modulo funcional agrupa sus propias clases. No hay capas tecnicas globales (un solo `service/`, un solo `repository/` para todo el proyecto).
2. **Estructura simple por defecto**: entity -> repository -> service -> controller. Sin puertos, sin interfaces de repositorio separadas, sin capa de dominio puro, a menos que la complejidad del modulo lo justifique.
3. **Reglas de negocio complejas se extraen localmente**: si un modulo tiene logica que merece testing aislado, se extrae a clases `Policy` o `Strategy` dentro del mismo feature. Sin puertos ni adaptadores.
4. **Adaptadores solo para integraciones externas**: pagos, IA e impresion se encapsulan detras de interfaces (porque la implementacion puede cambiar: mock vs real). No se hace lo mismo con repositorios JPA -- no vamos a cambiar de base de datos.
5. **Transaccionalidad explicita**: las operaciones que cruzan multiples tablas (stock + pedido + pago) se ejecutan en transacciones de base de datos manejadas a nivel de service.
6. **Validacion en dos capas**: los DTOs de entrada se validan con Bean Validation (`@Valid`) en el controlador, y las reglas de negocio se validan dentro del service.

---

## Estructura base de cada feature

El patron por defecto para cualquier modulo del sistema es:

```text
inventory/                    -- ejemplo: modulo de stock
  StockItem.java              -- Entidad JPA (con anotaciones)
  StockRepository.java        -- Spring Data JPA (extiende JpaRepository)
  StockService.java           -- Servicio con @Transactional y logica de negocio
  StockController.java        -- Controlador REST
  StockRequest.java           -- DTO de request
  StockResponse.java          -- DTO de response
```

Sin capas adicionales. Sin interfaces de repositorio separadas. El service usa el repository directo.

### Cuando se justifica extraer logica a clases separadas

Solo cuando hay reglas de negocio que:
- Son complejas (condicionales, calculos, algoritmos)
- Merecen tests unitarios aislados (sin Spring, sin BD)
- Se reutilizan entre distintos metodos del service

En ese caso, se agregan clases de dominio dentro del mismo feature:

```text
inventory/
  StockItem.java              -- Entidad JPA
  StockRepository.java        -- Spring Data JPA
  StockPolicy.java            -- Reglas de negocio: FEFO, calculo de disponibilidad
  ReservationPolicy.java      -- Ciclo de vida de reservas
  StockService.java           -- Orquesta Policy + Repository + transaccion
  StockController.java
```

Las `Policy` son clases Java puras (sin Spring, sin JPA). Se testean con JUnit sin necesidad de levantar el contexto.

**Por que no usar este patron en todos los modulos**: para modulos CRUD (proveedores, etiquetas, usuarios), la separacion domain/application/infrastructure no aporta beneficio real y solo agrega archivos. El service puede contener la logica directamente.

### Mapeo concreto: inventory (stock) -- el mas complejo

```text
inventory/
  model/
    StockItem.java            -- Entidad: producto + sucursal + cantidades
    StockMovement.java        -- Entidad: movimiento individual
    StockMovementType.java    -- Enum: INGRESO, VENTA, AJUSTE, MERMA, CONSUMO_INTERNO
    StockLot.java             -- Entidad: lote con vencimiento
    StockReservation.java     -- Entidad: reserva con ciclo de vida
  policy/
    StockCalculator.java      -- Calculo de stock_disponible (pura, testeable)
    FefoStrategy.java         -- Logica de seleccion de lote (First Expired First Out)
    ReservationPolicy.java    -- Reglas: cuando se crea/libera/confirma/venece una reserva
  repository/
    StockItemRepository.java         -- Spring Data JPA
    StockMovementRepository.java     -- Spring Data JPA
    StockLotRepository.java          -- Spring Data JPA
    StockReservationRepository.java  -- Spring Data JPA
  service/
    StockService.java         -- Orquesta policies + repositories + @Transactional
  web/
    StockController.java
    StockRequest.java
    StockResponse.java
```

**Por que este enfoque mixto**: para `inventory`, las reglas de FEFO, calculo de disponibilidad y ciclo de reservas son lo suficientemente complejas y criticas como para justificar testing unitario aislado. Pero no necesitan puertos, ni interfaces de repositorio, ni capa de aplicacion separada. Las policies son solo clases que el service llama. Simple, testeable, sin sobrediseno.

### Organizacion por feature vs capas horizontales

**Por que organizar por feature y no por capa tecnica** (todos los controllers en `controller/`, todos los services en `service/`):

En un proyecto que crece a 12+ modulos, la organizacion horizontal se vuelve inmanejable. Cada modulo con su propia estructura permite:

- Desarrollar modulos en paralelo sin conflictos de git
- Entender un modulo completo sin leer todo el codigo base
- Extraer un modulo a microservicio en el futuro si es necesario
- Probar cada modulo de forma aislada
- Navegar el proyecto por funcionalidad, no por tecnologia

---

## Arquitectura de transacciones

### Mapa de transacciones criticas

Cada operacion que involucra multiples tablas debe ejecutarse en una sola transaccion. Estas son las transacciones clave del sistema:

| Operacion | Tablas involucradas | Aislamiento | Estrategia |
|---|---|---|---|
| `CreateSale` (venta presencial) | `sales`, `sale_items`, `branch_stock`, `stock_movements`, `stock_lots` | READ COMMITTED + FOR UPDATE | Transaccion unica |
| `ConfirmPayment` (pago + reserva) | `payments`, `online_orders`, `stock_reservations`, `branch_stock` | REPEATABLE READ | Transaccion unica con revalidacion |
| `DeliverOrder` (entrega) | `stock_reservations`, `branch_stock`, `stock_lots`, `stock_movements` | READ COMMITTED + FOR UPDATE | Transaccion unica |
| `CancelOrder` | `online_orders`, `stock_reservations`, `branch_stock` | READ COMMITTED + FOR UPDATE | Transaccion unica |
| `RegisterStockEntry` | `branch_stock`, `stock_lots`, `stock_movements` | READ COMMITTED | Transaccion unica |

### Principio: una transaccion por operacion

Cada metodo de service que modifica datos lleva `@Transactional`. Las operaciones de solo lectura no llevan transaccion.

**Estrategia de bloqueo para stock**:

```text
INICIO TRANSACCION
  1. SELECT ... FROM branch_stock WHERE product_id=X AND branch_id=Y FOR UPDATE
     -- Esto bloquea la fila. Otros hilos esperan.
  2. Calcular stock_disponible = fisico - reservado
  3. Si stock_disponible >= cantidad_solicitada:
     a. INSERT reserva o UPDATE branch_stock
     b. INSERT stock_movement
     c. INSERT venta/pedido/pago
  4. Si stock_disponible < cantidad_solicitada:
     a. Lanzar InsufficientStockException (rollback automatico)
FIN TRANSACCION (COMMIT o ROLLBACK)
```

**Por que FOR UPDATE y no OPTIMISTIC LOCKING**: el optimistic locking con `@Version` funciona bien cuando las colisiones son raras y el costo de reintentar es bajo. Pero en este sistema:
- La ventana entre "el cliente ve stock" y "el cliente confirma" puede ser larga (minutos)
- Reintentar una transaccion de venta completa no es trivial
- El volumen es bajo (decenas de transacciones concurrentes, no miles)
- La certeza de no sobrevender es un requisito funcional, no una optimizacion

---

## Arquitectura de errores

### Excepciones de negocio

Cada modulo define sus propias excepciones, todas heredando de una base comun `DomainException` (RuntimeException):

| Modulo | Excepciones |
|---|---|
| `catalog` | `ProductNotFoundException`, `ProductNotPublishedException`, `InvalidBarcodeException` |
| `inventory` | `InsufficientStockException`, `LotExpiredException`, `ReservationExpiredException` |
| `orders` | `OrderNotFoundException`, `OrderInvalidStateException`, `OrderCannotBeCancelledException` |
| `payments` | `InvalidPaymentStateException`, `PaymentAlreadyConfirmedException` |
| `auth` | `InvalidCredentialsException`, `TokenExpiredException`, `AccountDisabledException` |

### Flujo de error completo

```text
Controlador recibe request invalido
    ↓ Bean Validation
MethodArgumentNotValidException
    ↓ @ControllerAdvice global
ApiError con code=VALIDATION_ERROR

O bien:

Service ejecuta @Transactional
    ↓ Regla de negocio violada
DomainException (runtime)
    ↓ Propaga naturalmente
@ControllerAdvice del modulo o global
    ↓
ApiError con codigo semantico (INSUFFICIENT_STOCK, etc.)

O bien:

Frontend recibe JSON estructurado
    ↓ Interceptor HTTP global decide:
      401 -> redirect a login
      403 -> toast "sin permisos"
      4xx -> toast con mensaje del backend
      5xx -> toast "error del servidor"
```

### Codigos de error por modulo

| Modulo | Codigos de error |
|---|---|
| `catalog` | `PRODUCT_NOT_FOUND`, `PRODUCT_NOT_PUBLISHED`, `CATEGORY_NOT_FOUND`, `INVALID_BARCODE` |
| `inventory` | `INSUFFICIENT_STOCK`, `LOT_EXPIRED`, `RESERVATION_EXPIRED` |
| `orders` | `ORDER_NOT_FOUND`, `ORDER_INVALID_STATE` |
| `payments` | `INVALID_PAYMENT_STATE`, `PAYMENT_ALREADY_CONFIRMED` |
| `auth` | `INVALID_CREDENTIALS`, `TOKEN_EXPIRED`, `ACCOUNT_DISABLED` |

---

## Arquitectura de seguridad

### Flujo de autenticacion JWT

```text
Cliente
  │
  POST /api/auth/login { email, password }
  │
  ├── 1. Backend valida credenciales contra DB (BCrypt)
  ├── 2. Backend genera JWT con:
  │       - sub: user_id
  │       - roles: [ADMIN_GENERAL, ENCARGADO_SUCURSAL]
  │       - branch_id: si aplica
  │       - iat, exp (24h)
  ├── 3. Backend devuelve { token, refresh_token, user }
  │
  Cliente almacena token en memoria/localStorage
  │
  GET /api/admin/products (Header: Authorization: Bearer <token>)
  │
  ├── 1. Filtro JWT extrae y valida token
  ├── 2. Filtro setea SecurityContext con usuario autenticado
  ├── 3. Controlador ejecuta (con @PreAuthorize si es necesario)
  │
  Response (o 401 si token invalido/expirado)
```

### Decisiones de la arquitectura JWT

| Decision | Por que |
|---|---|
| **Access token de corta duracion (24h)** + **Refresh token de larga duracion (7 dias)** | Permite cerrar sesion forzadamente invalidando el refresh token. El access token corto limita el dano si se filtra. |
| **JWT no almacenado en servidor** | Stateless: no hay sesion en backend, facil escalar horizontalmente. La contrapartida es que no se puede invalidar un token antes de que expire (salvo con blacklist en Redis, que esta fuera del MVP). |
| **branch_id en claims** | Permite autorizar acceso a recursos de una sucursal especifica sin consultar DB en cada request. |
| **Refresh token en cookie HttpOnly** | Mas seguro que localStorage para tokens de larga duracion. El access token puede ir en localStorage ya que es corto. |

### Cadena de filtros Spring Security

```text
SecurityFilterChain
  ├── 1. CorsFilter                  -- Permite origenes configurados
  ├── 2. CsrfFilter (DISABLED)       -- API REST sin CSRF (stateless JWT)
  ├── 3. JwtAuthenticationFilter     -- Extrae JWT, valida, setea contexto
  ├── 4. ExceptionHandlerFilter      -- Captura errores de autenticacion
  │
  Rutas publicas: /api/public/**, /api/auth/login, /uploads/**
  Rutas admin: /api/admin/** (requiere autenticacion)
```

### Matriz de autorizacion (RBAC)

Cada endpoint se protege con `@PreAuthorize`:

```text
@PreAuthorize("hasRole('ADMIN_GENERAL')")
@PreAuthorize("hasRole('ENCARGADO_SUCURSAL') and @security.canAccessBranch(#branchId)")
@PreAuthorize("hasRole('EMPLEADO') and @security.isAssignedToBranch(#sucursalId)")
```

---

## Arquitectura de integraciones (adaptadores)

### Patron de adaptador

Todas las integraciones externas siguen el mismo patron. Esto SI es un caso donde vale la pena tener interfaces separadas, porque la implementacion va a cambiar (mock en MVP, real en produccion):

```text
Service del modulo
  └── Interface (puerto): PaymentGateway
      └── Implementacion Mock: MockPaymentGateway (MVP)
      └── Implementacion Real: MercadoPagoGateway (futuro)
```

### Mapa de integraciones

| Integracion | Interface | Mock MVP | Real futuro |
|---|---|---|---|
| Pagos | `PaymentGateway` | `MockPaymentGateway` (simula crear link, confirmar pago) | `MercadoPagoGateway` |
| IA | `AssistantPort` | `RuleBasedAssistantAdapter` (filtros BD + ranking) | `LlmAssistantAdapter` + `HybridRecommendationService` |
| Impresion | `LabelPrintingPort` | `MockLabelPrinter` (simula impresion, genera PDF) | Driver de impresora termica Zebra/Epson |
| Archivos | `FileStoragePort` | `LocalFileStorageAdapter` (sistema de archivos) | `S3FileStorageAdapter` (Cloudinary / AWS S3) |

**Por que este patron si y no para repositorios JPA**: porque las integraciones externas van a cambiar de implementacion (mock -> real). Los repositorios JPA no -- siempre van a ser JPA contra PostgreSQL. Agregar una interfaz separada para cada repositorio "por si cambiamos de BD" es abstraccion paga sin beneficio real.

---

## Arquitectura de almacenamiento de archivos

### Estrategia MVP: sistema de archivos local

```text
uploads/
  products/
    {productId}/
      1.jpg      -- Imagen principal
      2.jpg      -- Opcional
      3.jpg      -- Opcional
  labels/
    {jobId}.pdf  -- Etiquetas generadas
```

**Por que local y no S3 en el MVP**: para una tesis academica o un negocio pequeno, el volumen de imagenes es bajo (cientos, no miles). El sistema de archivos local es simple, no requiere dependencias externas y es suficiente para demostrar el flujo completo. Nginx configurado para servir archivos estaticos desde `uploads/` da el rendimiento necesario.

### Estrategia de migracion futura

Cuando se necesite escalar (multiples servidores, alta disponibilidad), se reemplaza `LocalFileStorageAdapter` por `S3FileStorageAdapter` implementando el mismo `FileStoragePort`. Las URLs de imagenes pasan de `/uploads/products/{id}.jpg` a URLs de Cloudinary/S3.

---

## Arquitectura de scheduling y tareas asincronicas

### Tareas programadas (cron)

| Tarea | Frecuencia | Proposito |
|---|---|---|
| Liberar reservas vencidas | Post-MVP | En MVP la reserva se crea post-pago, por lo tanto no vence. Se libera solo al cancelar o entregar. |
| Alertas de vencimiento | Cada 24h | Detectar lotes con `expiration_date < NOW() + 30d` y generar notificaciones |
| Sugerencias de reposicion | Cada 24h | Ejecutar logica de bajo stock y actualizar dashboard |

**Implementacion**: `@Scheduled` de Spring Boot para el MVP. Las tareas se ejecutan en el mismo proceso. Si en el futuro se necesita escalar, se migran a un scheduler externo (Quartz, job service separado).

### Por que no usar mensajeria (RabbitMQ, Kafka) en el MVP

Para el volumen y la complejidad de este sistema, la mensajeria asincronica agregaria complejidad operativa sin beneficio real. Todas las operaciones criticas son sincronas dentro de una transaccion. Las tareas programadas cubren los casos de limpieza/alertas que no necesitan respuesta inmediata.

---

## Arquitectura de testing

### Piramide de tests

```text
         ╱╲
        ╱  ╲
       ╱ E2E ╲          -- 3-5 flujos criticos (Playwright/Cypress)
      ╱────────╲
     ╱  Integration ╲   -- Testcontainers + Spring Boot Test
    ╱────────────────╲  -- Flujos transaccionales completos
   ╱   Unit (Policies) ╲ -- Reglas de negocio puras, sin Spring
  ╱──────────────────────╲-- Solo donde hay clases Policy/Strategy
```

### Reglas de testing

1. **Policies**: tests unitarios puros. Las reglas de negocio (calculo de stock disponible, FEFO, cambios de estado) se prueban sin Spring, sin base de datos. Son los tests mas valiosos del sistema.
2. **Services**: tests de integracion con Testcontainers para flujos transaccionales. Verificar que una venta + descuento de stock es atomica.
3. **Controllers**: tests de slice `@WebMvcTest` con mock de servicios.
4. **Adaptadores**: tests de integracion especificos para cada integracion externa.

**Por que Testcontainers y no H2**: H2 no soporta `FOR UPDATE` de la misma forma que PostgreSQL, y las diferencias en dialecto SQL pueden ocultar bugs. Testcontainers levanta una instancia real de PostgreSQL en Docker para cada ejecucion de tests. Es mas lento pero mucho mas confiable.

---

## Mapa de dependencias entre modulos

```text
auth       -- Sin dependencias de otros modulos
companies  -- Depende de: auth
catalog    -- Depende de: companies, auth
inventory  -- Depende de: catalog, companies, auth
sales      -- Depende de: catalog, inventory, auth
orders     -- Depende de: catalog, inventory, auth
           -- Incluye payment como submodulo (integra PaymentGateway interface)
payments   -- Submodulo de orders, no es modulo independiente en MVP
suppliers  -- Depende de: catalog, auth
labels     -- Depende de: catalog, auth
analytics  -- Depende de: catalog, inventory, sales, orders, auth
assistant  -- Depende de: catalog, inventory, suppliers, auth
```

### Orden propuesto de desarrollo

```text
Iteracion 1: auth + companies
Iteracion 2: catalog
Iteracion 3: inventory
Iteracion 4: sales (presencial)
Iteracion 5: payments + orders
Iteracion 6: suppliers + labels
Iteracion 7: analytics
Iteracion 8: assistant
```

**Por que este orden**: cada iteracion construye sobre la anterior. No se puede manejar stock sin productos (catalog), no se puede vender sin stock (inventory), no se puede hacer pedidos sin pagos. El orden minimiza el riesgo de tener que redisenar modulos previos.

---

> [!seealso] Notas relacionadas
> - [[Vision General]] -- stack tecnologico y decision de monolito modular
> - [[Base de Datos]] -- esquema y transacciones
> - [[Endpoints API]] -- lista de endpoints REST
> - [[Testing]] -- estrategia de pruebas
> - [[07 - Referencias/Despliegue]] -- Docker Compose
> - Volver a [[_Index]]
