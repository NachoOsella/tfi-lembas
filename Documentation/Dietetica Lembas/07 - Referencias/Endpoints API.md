---
title: "Diseno de la API REST"
tags:
  - referencia
  - api
  - endpoints
  - rest
  - arquitectura-detallada
---

# Diseno de la API REST

> [!info] Arquitectura completa de la API REST: diseno de recursos, contratos de request/response, paginacion, versionado, ejemplos concretos.

---

## Principios de diseno de la API

1. **REST nominal**: recursos identificados por sustantivos en plural (`/products`, `/orders`). Acciones representadas por metodos HTTP (GET, POST, PUT, PATCH, DELETE).
2. **Dos espacios de URLs**: `/api/public/` para endpoints accesibles sin autenticacion (catalogo, asistente, tracking). `/api/admin/` para endpoints protegidos del backoffice.
3. **Formato unico de respuesta**: toda respuesta exitosa devuelve el recurso directamente. Toda respuesta de error sigue el formato `ApiError`.
4. **Versionado por prefijo de ruta**: `/api/v1/` preparado para el futuro aunque inicialmente solo haya una version.
5. **Paginacion consistente**: todos los listados usan el mismo esquema de paginacion (offset + limit o cursor).
6. **HATEOAS minimo**: las respuestas incluyen links a recursos relacionados cuando es relevante (opcional en MVP).

---

## Formato de respuestas

### Respuesta exitosa

```json
// GET /api/admin/products/123
{
  "id": 123,
  "name": "Granola 500g",
  "barcode": "7791234567890",
  "category": { "id": 5, "name": "Cereales" },
  "brand": { "id": 3, "name": "NaturalLife" },
  "tags": ["sin-tacc", "vegano", "sin-azucar"],
  "price": 4900.00,
  "onlineStatus": "PUBLISHED",
  "images": [
    { "id": 1, "url": "/uploads/products/123/1.jpg", "main": true }
  ],
  "stock": [
    { "branchId": 1, "branchName": "Centro", "available": 10 },
    { "branchId": 2, "branchName": "Nueva Cordoba", "available": 3 }
  ],
  "createdAt": "2026-05-01T10:00:00Z",
  "updatedAt": "2026-05-12T15:30:00Z"
}
```

### Respuesta paginada

```json
// GET /api/admin/products?page=1&size=20
{
  "data": [ ...array de productos... ],
  "pagination": {
    "page": 1,
    "size": 20,
    "total": 156,
    "totalPages": 8,
    "hasNext": true,
    "hasPrev": false
  }
}
```

**Por que paginacion offset y no cursor**: es la mas simple de implementar y entender. Para una dietetica con cientos de productos (no millones), el rendimiento es aceptable. Si en el futuro el catalogo creciera a decenas de miles, se migraria a cursor-based pagination.

### Respuesta de error

```json
{
  "status": 409,
  "code": "INSUFFICIENT_STOCK",
  "message": "Stock insuficiente para el producto Granola 500g",
  "details": {
    "productId": 123,
    "available": 2,
    "requested": 5,
    "branchId": 1
  },
  "timestamp": "2026-05-12T15:30:00Z",
  "path": "/api/admin/orders",
  "traceId": "abc-123-def"
}
```

**Los campos `details` varian segun el error**:
- `VALIDATION_ERROR`: `{ field: "price", rejectedValue: -100, message: "must be greater than 0" }`
- `PRODUCT_NOT_FOUND`: `{ productId: 999 }`
- `ORDER_INVALID_STATE`: `{ orderId: 42, currentState: "ENTREGADO", requestedAction: "cancel" }`
- `INSUFFICIENT_STOCK`: `{ productId: 123, available: 2, requested: 5 }`

---

## Contratos de recursos principales

### Auth

```
POST /api/auth/login
  Request:  { email: string, password: string }
  Response: { token: string, refreshToken: string, user: UserDto }
  Errors:   INVALID_CREDENTIALS (401), ACCOUNT_DISABLED (403)

POST /api/auth/refresh
  Request:  { refreshToken: string }
  Response: { token: string, refreshToken: string }
  Errors:   TOKEN_EXPIRED (401), TOKEN_INVALID (401)

POST /api/auth/logout
  Headers:  Authorization: Bearer <token>
  Response: 204 No Content

GET /api/auth/me
  Headers:  Authorization: Bearer <token>
  Response: { id, email, name, role, branchId, branchName }
  Errors:   UNAUTHORIZED (401)
```

### Catalogo publico

```
GET /api/public/catalog/products?q=&categoryId=&tag=&branchId=&page=&size=
  Response: PaginatedResponse<ProductSummaryDto>
  Notas:    Solo productos con onlineStatus=PUBLISHED. branchId filtra por stock disponible.
            Si no se especifica branchId, no se muestra stock (solo "disponible" generico).

GET /api/public/catalog/products/{id}?branchId=
  Response: ProductDetailDto
  Errors:   PRODUCT_NOT_FOUND (404)
  Notas:    Incluye stock disponible en la sucursal solicitada.

GET /api/public/catalog/categories
  Response: [ { id, name, productCount } ]

GET /api/public/catalog/products/{id}/recommendations?branchId=
  Response: [ ProductSummaryDto ]
  Notas:    Productos similares (misma categoria/etiquetas) con stock en la sucursal.
```

### Pedidos (publico)

```
POST /api/public/orders
  Request:  {
              items: [ { productId: 123, quantity: 2 } ],
              branchId: 1,
              customerName: string,
              customerEmail?: string,
              customerPhone: string,
              deliveryType: "PICKUP" | "SHIPPING",
              deliveryData?: { address, city, phone, notes }
            }
  Response: OrderCreatedDto { orderId, orderNumber, paymentLink, estimatedTotal }
  Errors:   INSUFFICIENT_STOCK (409), PRODUCT_NOT_FOUND (404), BRANCH_INVALID (400)
  Notas:    Checkout invitado: no se requiere autenticacion ni registro.
            No se reserva stock aqui. Solo se valida disponibilidad.
            El stock se reserva al confirmar el pago.

GET /api/public/orders/{orderNumber}/tracking
  Response: OrderTrackingDto { orderNumber, status, estimatedDate, timeline }
  Notas:    Sin autenticacion. Cualquiera con el numero de pedido puede tracking.
```

### Productos (admin)

```
GET /api/admin/products?q=&categoryId=&onlineStatus=&page=&size=
  Response: PaginatedResponse<ProductSummaryDto>
  Notas:    Incluye productos en todos los estados (borrador, publicado, pausado).

POST /api/admin/products
  Request:  {
              name, barcode?, categoryId, brandId?, tags?: [],
              price: number, onlineStatus?: string,
              stockInitial?: [ { branchId: 1, quantity: 10 } ]
            }
  Response: ProductDetailDto (201 Created)
  Errors:   VALIDATION_ERROR (400), BARCODE_DUPLICATED (409)

GET /api/admin/products/{id}
  Response: ProductDetailDto
  Errors:   PRODUCT_NOT_FOUND (404)

PUT /api/admin/products/{id}
  Request:  Mismos campos que POST (todos opcionales en PUT parcial via PATCH)
  Response: ProductDetailDto
  Errors:   PRODUCT_NOT_FOUND (404), VALIDATION_ERROR (400)

PATCH /api/admin/products/{id}/publish
  Response: ProductDetailDto
  Errors:   PRODUCT_NOT_FOUND (404), INVALID_STATE (409, si ya esta publicado)

PATCH /api/admin/products/{id}/pause
  Response: ProductDetailDto
  Errors:   PRODUCT_NOT_FOUND (404), INVALID_STATE (409, si no estaba publicado)

DELETE /api/admin/products/{id}
  Response: 204 No Content
  Errors:   PRODUCT_NOT_FOUND (404), PRODUCT_HAS_ORDERS (409)
```

### Stock (admin)

```
GET /api/admin/stock?productId=&branchId=&lowStock=&expiringSoon=
  Response: [ BranchStockDto ]

POST /api/admin/stock/entries
  Request:  { productId, branchId, quantity, lotExpiration?: date, supplierId?, cost?: number }
  Response: StockMovementDto
  Notas:    Crea ingreso de stock. Si tiene vencimiento, crea lote.

POST /api/admin/stock/adjustments
  Request:  { productId, branchId, quantity (positivo=sube, negativo=baja), reason }
  Response: StockMovementDto
  Errors:   INSUFFICIENT_STOCK (409, si quantity > disponible)

POST /api/admin/stock/internal-consumptions
  Request:  { productId, branchId, quantity, reason, employeeId }
  Response: StockMovementDto

POST /api/admin/stock/waste
  Request:  { productId, branchId, quantity, reason, lotId? }
  Response: StockMovementDto

GET /api/admin/stock/movements?productId=&branchId=&type=&from=&to=&page=&size=
  Response: PaginatedResponse<StockMovementDto>

GET /api/admin/stock/low-stock?branchId=&threshold=
  Response: [ LowStockAlertDto { product, branch, current, minimum, suggestedOrder } ]

GET /api/admin/stock/expiring-soon?branchId=&days=
  Response: [ ExpiringLotAlertDto { lot, product, branch, expirationDate, quantity } ]
```

### Ventas presenciales (admin)

```
POST /api/admin/sales
  Request:  {
              items: [ { productId: 123, quantity: 2, unitPrice: 4900 } ],
              branchId: 1,
              paymentMethod: "CASH" | "TRANSFER" | "QR",
              employeeId: 42
            }
  Response: SaleCompletedDto (201 Created)
  Errors:   INSUFFICIENT_STOCK (409), PRODUCT_NOT_FOUND (404),
            BRANCH_MISMATCH (400, si empleado no pertenece a la sucursal)
  Notas:    Operacion transaccional. Valida stock, descuenta, registra venta.
            branchId se obtiene del empleado autenticado (no del request).

GET /api/admin/sales?from=&to=&branchId=&employeeId=&page=&size=
  Response: PaginatedResponse<SaleSummaryDto>

GET /api/admin/sales/{id}
  Response: SaleDetailDto
```

### Pedidos (admin)

```
GET /api/admin/orders?status=&branchId=&from=&to=&page=&size=
  Response: PaginatedResponse<OrderSummaryDto>

GET /api/admin/orders/{id}
  Response: OrderDetailDto

PATCH /api/admin/orders/{id}/mark-paid
  Request:  { paymentId, externalReference? }
  Response: OrderDetailDto
  Errors:   ORDER_INVALID_STATE (409, si no esta PENDIENTE_PAGO)
  Notas:    Revalida stock y precio antes de marcar como pagado.

PATCH /api/admin/orders/{id}/prepare
  Response: OrderDetailDto
  Errors:   ORDER_INVALID_STATE (409)

PATCH /api/admin/orders/{id}/ready-for-pickup
  Response: OrderDetailDto
  Errors:   ORDER_INVALID_STATE (409)

PATCH /api/admin/orders/{id}/out-for-delivery
  Response: OrderDetailDto
  Errors:   ORDER_INVALID_STATE (409)

PATCH /api/admin/orders/{id}/delivered
  Response: OrderDetailDto
  Errors:   ORDER_INVALID_STATE (409)
  Notas:    Confirma reserva, descuenta stock definitivamente (FEFO), registra movimiento.

PATCH /api/admin/orders/{id}/cancel
  Request:  { reason }
  Response: OrderDetailDto
  Errors:   ORDER_INVALID_STATE (409, si ya esta ENTREGADO o CANCELADO)
  Notas:    Libera reserva de stock si existe.
```

### Proveedores (admin)

```
GET  /api/admin/suppliers?q=&page=&size=
  Response: PaginatedResponse<SupplierDto>

POST /api/admin/suppliers
  Request:  { name, contactName?, phone?, email?, cuit? }
  Response: SupplierDto (201 Created)

PUT /api/admin/suppliers/{id}
  Request:  { ... campos editables ... }
  Response: SupplierDto

PATCH /api/admin/products/{id}/supplier-cost
  Request:  { supplierId, cost }
  Response: ProductDetailDto
  Notas:    Ingreso manual de costo por producto-proveedor. El sistema sugiere
            precio de venta basado en el margen configurado y registra en historial.
            No hay flujo de aprobacion en MVP; el administrador aplica directamente.
  Errors:   VALIDATION_ERROR (400, si el costo es invalido)
```

### Imagenes de producto (admin)

```
POST /api/admin/products/{id}/images
  Request:  multipart/form-data (file: imagen, main?: boolean)
  Response: ImageDto (201 Created)
  Errors:   IMAGE_TOO_LARGE (400), INVALID_IMAGE_FORMAT (400)

GET /api/admin/products/{id}/images
  Response: [ ImageDto ]

DELETE /api/admin/products/{id}/images/{imageId}
  Response: 204 No Content

PATCH /api/admin/products/{id}/images/{imageId}/main
  Response: ImageDto
```

### Analytics (admin)

```
GET /api/admin/analytics/dashboard?branchId=
  Response: DashboardDto {
              salesToday, salesTodayChange,
              pendingOrders, pendingOrdersChange,
              lowStockItems, expiringSoonItems,
              topProducts: [ { product, quantity, revenue } ],
              recentOrders: [ OrderSummaryDto ]
            }

GET /api/admin/analytics/sales?from=&to=&branchId=&groupBy=day|week|month
  Response: [ { period, total, count, averageTicket } ]

GET /api/admin/analytics/products?from=&to=&branchId=&sortBy=quantity|revenue|margin
  Response: [ ProductAnalyticsDto ]

GET /api/admin/analytics/inventory?branchId=
  Response: { lowStock: [], expiringSoon: [], totalProducts, totalValue }

GET /api/admin/analytics/orders?from=&to=&branchId=
  Response: { byStatus: { PAGADO: 5, EN_PREPARACION: 3, ... }, avgPreparationTime }
```

### Asistente IA

```
POST /api/public/assistant/recommendations
  Request:  { query: "algo dulce sin azucar", branchId: 1, limit?: 5 }
  Response: {
              recommendations: [ ProductSummaryDto ],
              alternatives: [ ProductSummaryDto ] | null,
              message: "Encontre estas opciones..." | "No tengo stock de eso, pero..."
            }
  Errors:   INVALID_QUERY_FORMAT (400), NO_RECOMMENDATIONS_FOUND (404)
  Notas:    No usa LLM. Filtra por tags + stock. Si no hay resultados,
            busca alternativas en la misma categoria.

POST /api/admin/assistant/restock-suggestions?branchId=&limit=
  Response: [ { product, currentStock, minimumStock, salesVelocity, suggestedQuantity } ]

POST /api/admin/assistant/promotion-suggestions?branchId=&productId=
  Response: { product, currentPrice, suggestedDiscount, suggestedPrice, reason, urgency }
```

### Etiquetas (admin)

```
POST /api/admin/labels/price-tags
  Request:  { productIds: [123, 124, 125], branchId?: number, includeDate?: boolean }
  Response: LabelJobDto { jobId, url: "/uploads/labels/{jobId}.pdf" }
```

---

## Versionado de la API

**Estrategia**: prefijo de ruta (`/api/v1/`).

```text
/api/v1/public/catalog/products
/api/v1/admin/orders
```

**Por que prefijo de ruta y no header**: es visible, facil de depurar, y no requiere negociacion de contenido. El header `Accept-Version` es mas "puro" REST pero menos practico.

**Cuando versionar**: cuando un cambio en la respuesta rompe clientes existentes. Agregar campos nuevos a una respuesta no requiere nueva version. Cambiar el tipo de un campo existente o eliminar un campo si requiere nueva version.

Para el MVP, no se implementa versionado activo. La ruta base es `/api/` y se agregara `/api/v1/` cuando haya necesidad de mantener compatibilidad hacia atras.

---

## Rate limiting y seguridad de API

Para el MVP, el rate limiting se implementa a nivel de `application.yml` con configuracion simple:

```yaml
# Por defecto, Spring Boot no incluye rate limiting.
# Se puede implementar con un filtro simple o bucket4j.
```

**Endpoints publicos con restriccion**:

| Endpoint | Limite sugerido | Justificacion |
|---|---|---|
| `POST /api/auth/login` | 5 intentos por minuto por IP | Prevenir fuerza bruta |
| `POST /api/public/assistant/recommendations` | 30 por minuto por IP | Evitar abuso del asistente |
| `GET /api/public/catalog/products` | 60 por minuto por IP | Carga normal de catalogo |

---

## Estrategia de documentacion de API

Para el MVP, se usa **SpringDoc OpenAPI** (swagger) que genera documentacion interactiva automaticamente desde las anotaciones del codigo.

```yaml
# Dependencia: springdoc-openapi-starter-webmvc-ui
# URL: /swagger-ui.html
```

La documentacion incluye:
- Todos los endpoints con parametros
- Schemas de request/response
- Codigos de error por endpoint
- Posibilidad de probar endpoints desde el navegador
- Autenticacion JWT configurable en Swagger UI

---

> [!seealso] Notas relacionadas
> - [[Backend]] -- implementacion de controladores y casos de uso
> - [[Testing]] -- tests de API con MockMvc
> - [[Despliegue]] -- configuracion de CORS y Nginx
> - Volver a [[_Index]]
