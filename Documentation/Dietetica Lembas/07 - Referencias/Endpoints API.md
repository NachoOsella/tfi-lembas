---
title: "Diseno de la API REST"
tags:
  - referencia
  - api
  - endpoints
  - rest
---

# Diseno de la API REST (MVP)

> [!info] Endpoints para tienda online con Mercado Pago Checkout Pro, caja operativa con arqueo de efectivo, y ventas presenciales.

---

## Principios

1. **REST nominal**: recursos en plural (`/products`, `/orders`).
2. **Cuatro espacios de URLs**: `/api/store/` (publico), `/api/customer/` (rol CUSTOMER), `/api/admin/` (ADMIN/MANAGER/EMPLOYEE), `/api/webhooks/` (externo).
3. **Formato unico**: respuestas exitosas directas, errores con `ApiError`.
4. **Paginacion**: offset + limit (`page`, `size`).

---

## Formato de respuestas

### Exito

```json
// GET /api/admin/products/123
{
  "id": 123,
  "name": "Granola 500g",
  "barcode": "7791234567890",
  "category": { "id": 5, "name": "Cereales" },
  "brandName": "NaturalLife",
  "salePrice": 4900.00,
  "onlineStatus": "PUBLISHED",
  "imageUrl": "/uploads/products/123.jpg",
  "createdAt": "2026-05-01T10:00:00Z"
}
```

### Error

```json
{
  "status": 409,
  "code": "INSUFFICIENT_STOCK",
  "message": "Stock insuficiente para el producto Granola 500g",
  "details": { "productId": 123, "available": 2, "requested": 5, "branchId": 1 },
  "timestamp": "2026-05-12T15:30:00Z",
  "path": "/api/customer/orders"
}
```

---

## Contratos

### Auth (publico)

```
POST /api/auth/register
  Request:  { firstName, lastName, email, password, phone? }
  Response: { token, refreshToken, user }
  Errors:   EMAIL_DUPLICATED (409), VALIDATION_ERROR (400)
  Notas:    Crea usuario con rol CUSTOMER.

POST /api/auth/login
  Request:  { email, password }
  Response: { token, refreshToken, user }
  Errors:   INVALID_CREDENTIALS (401), ACCOUNT_DISABLED (403)

GET /api/auth/me
  Headers:  Authorization: Bearer <token>
  Response: { id, email, firstName, lastName, role, branchId, branchName }
```

### Catalogo publico (store)

```
GET /api/store/products?q=&categoryId=&branchId=&page=&size=
  Response: PaginatedResponse<ProductSummaryDto>
  Notas:    Solo onlineStatus=PUBLISHED.

GET /api/store/products/{id}?branchId=
  Response: ProductDetailDto
  Errors:   PRODUCT_NOT_FOUND (404)

GET /api/store/categories
  Response: [ { id, name, productCount } ]
```

### Perfil de cliente (customer)

```
GET /api/customer/profile
  Headers:  Authorization: Bearer <token>
  Response: { id, firstName, lastName, email, phone, createdAt }

PATCH /api/customer/profile
  Headers:  Authorization: Bearer <token>
  Request:  { firstName?, lastName?, phone? }
  Response: { id, firstName, lastName, email, phone }
```

### Pedidos online con Mercado Pago (customer)

El carrito es local (localStorage). Al confirmar, se crea la orden y la preferencia de MP.

```
POST /api/customer/orders
  Headers:  Authorization: Bearer <token>
  Request:  {
              items: [ { productId: 123, quantity: 2 } ],
              paymentMethod: "MERCADO_PAGO"
            }
  Response: OrderCreatedDto { id, orderNumber, status: "PENDING_PAYMENT", total }
  Errors:   INSUFFICIENT_STOCK (409), PRODUCT_NOT_FOUND (404), UNAUTHORIZED (401)
  Notas:    Valida stock, crea order (type=ONLINE, status=PENDING_PAYMENT),
            crea payment (provider=MERCADO_PAGO, method=CHECKOUT_PRO, status=PENDING),
            NO descuenta stock aun.

GET /api/customer/orders
  Headers:  Authorization: Bearer <token>
  Response: [ OrderSummaryDto ]
  Notas:    Solo ordenes del CUSTOMER autenticado. Incluye estado del payment.

GET /api/customer/orders/{id}
  Headers:  Authorization: Bearer <token>
  Response: OrderDetailDto (con payments incluidos)
  Errors:   ORDER_NOT_FOUND (404), FORBIDDEN (403)
```

### Checkout Mercado Pago (customer)

```
POST /api/customer/orders/{orderId}/checkout/mp
  Headers:  Authorization: Bearer <token>
  Response: { initPoint: string, preferenceId: string }
  Errors:   ORDER_NOT_FOUND (404), ORDER_INVALID_STATE (409)
  Notas:    Crea preferencia en MP, guarda provider_preference_id en payment,
            devuelve init_point para redirigir al checkout de MP.
            Este endpoint es idempotente: si ya hay preferencia, la devuelve.
```

### Webhook Mercado Pago (publico, firmado)

```
POST /api/webhooks/mercadopago
  Headers:  X-Signature: <firma MP>
  Response: 200 OK
  Notas:    Recibe notificacion de MP. Verifica firma.
            Consulta estado real del pago en MP.
            Si APPROVED: actualiza payment, descuenta stock FEFO,
            cambia order a PAID, registra stock_movements.
            Si REJECTED: actualiza payment, order a PAYMENT_FAILED.
            Procesamiento con idempotencia (por provider_payment_id).
```

### Productos (admin)

```
GET /api/admin/products?q=&categoryId=&onlineStatus=&page=&size=
  Response: PaginatedResponse<ProductSummaryDto>

POST /api/admin/products
  Request:  { name, barcode?, categoryId, brandName?, salePrice, onlineStatus?, imageUrl? }
  Response: ProductDetailDto (201)

PUT /api/admin/products/{id}
  Request:  { name?, barcode?, categoryId?, brandName?, salePrice?, onlineStatus?, imageUrl? }
  Response: ProductDetailDto

PATCH /api/admin/products/{id}/status
  Request:  { onlineStatus: "PUBLISHED" | "PAUSED" | "DRAFT" }
  Response: ProductDetailDto

DELETE /api/admin/products/{id}
  Response: 204
```

### Stock (admin)

```
GET /api/admin/stock/lots?productId=&branchId=&expiringSoon=
  Response: [ StockLotDto ]

POST /api/admin/stock/lots
  Request:  { productId, branchId, quantity, lotCode?, expirationDate?, costPrice? }
  Response: StockLotDto (201)

POST /api/admin/stock/adjustments
  Request:  { productId, branchId, quantity, reason, stockLotId? }
  Response: StockMovementDto
  Errors:   INSUFFICIENT_STOCK (409)

GET /api/admin/stock/movements?productId=&branchId=&type=&from=&to=&page=&size=
  Response: PaginatedResponse<StockMovementDto>
```

### Caja (admin)

```
POST /api/admin/cash-sessions/open
  Request:  { openingCashAmount: number, openingNotes?: string }
  Response: CashSessionDto (201)
  Errors:   CASH_SESSION_ALREADY_OPEN (409)
  Notas:    Abre caja para la sucursal del usuario autenticado.
            Solo una caja abierta por sucursal.

GET /api/admin/cash-sessions/current
  Response: CashSessionDto | null
  Notas:    Devuelve la caja abierta actual de la sucursal del usuario.

POST /api/admin/cash-sessions/{id}/movements
  Request:  { type: "CASH_IN" | "CASH_OUT" | "ADJUSTMENT", method: "CASH" | "TRANSFER" | "OTHER", amount, reason }
  Response: CashMovementDto (201)
  Errors:   CASH_SESSION_NOT_OPEN (409)
  Notas:    Solo permite movimientos si la caja esta OPEN.

POST /api/admin/cash-sessions/{id}/close
  Request:  { countedCashAmount: number, closingNotes?: string, cashDifferenceReason?: string }
  Response: CashSessionDto
  Errors:   CASH_SESSION_NOT_OPEN (409), DIFFERENCE_REASON_REQUIRED (422)
  Notas:    Calcula expectedCashAmount. Si countedCashAmount != expectedCashAmount,
            exige cashDifferenceReason. No bloquea el cierre, pero obliga explicacion.
            Actualiza status a CLOSED.

GET /api/admin/cash-sessions
  Response: [ CashSessionSummaryDto ]
  Notas:    Historial de cierres de caja.

GET /api/admin/cash-sessions/{id}
  Response: CashSessionDetailDto
  Notas:    Incluye: apertura, cierre, ventas asociadas, payments,
            totales por metodo de pago, efectivo esperado vs contado, diferencia.
```

### Ventas presenciales (admin) -- POS

Requiere caja abierta en la sucursal. Crea orden POS con payment asociado a la caja.

```
POST /api/admin/pos/sales
  Request:  {
              items: [ { productId: 123, quantity: 2 } ],
              paymentMethod: "CASH" | "QR" | "TRANSFER" | "DEBIT_CARD" | "CREDIT_CARD",
              customerUserId?: number,
              customerNameSnapshot?: string
            }
  Response: OrderDetailDto (201)
  Errors:   INSUFFICIENT_STOCK (409), CASH_SESSION_REQUIRED (400),
            PRODUCT_NOT_FOUND (404)
  Notas:    Operacion transaccional:
            1. Valida caja abierta en la sucursal del empleado
            2. Valida stock (FOR UPDATE)
            3. Descuenta stock FEFO
            4. INSERT stock_movement (POS_SALE)
            5. INSERT order (type=POS, status=PAID)
            6. INSERT order_items (con snapshots)
            7. INSERT payment (cash_session_id, provider=MANUAL, method=segun request, status=APPROVED)
```

### Ordenes (admin)

```
GET /api/admin/orders?status=&branchId=&type=&from=&to=&page=&size=
  Response: PaginatedResponse<OrderSummaryDto>

GET /api/admin/orders/{id}
  Response: OrderDetailDto (con payments)

PATCH /api/admin/orders/{id}/mark-paid  → obsoleto. Usar webhook MP para ONLINE.
  Notas:   Solo aplica para ordenes POS (ya se crean pagadas).
           Para ONLINE, el pago se confirma via webhook de MP.

PATCH /api/admin/orders/{id}/prepare
  Response: OrderDetailDto
  Errors:   ORDER_INVALID_STATE (409)

PATCH /api/admin/orders/{id}/ready
  Response: OrderDetailDto
  Errors:   ORDER_INVALID_STATE (409)

PATCH /api/admin/orders/{id}/delivered
  Response: OrderDetailDto
  Errors:   ORDER_INVALID_STATE (409)
  Notas:    Solo cambia estado. NO descuenta stock.

PATCH /api/admin/orders/{id}/cancel
  Request:  { reason: string }
  Response: OrderDetailDto
  Errors:   ORDER_INVALID_STATE (409)
  Notas:    Revierte stock si la orden estaba pagada.
```

### Proveedores (admin)

```
GET /api/admin/suppliers?q=&page=&size=
  Response: PaginatedResponse<SupplierDto>

POST /api/admin/suppliers
  Request:  { name, contactName?, phone?, email?, cuit? }
  Response: SupplierDto (201)

PUT /api/admin/suppliers/{id}
  Request:  { name?, contactName?, phone?, email?, cuit? }
  Response: SupplierDto

GET /api/admin/supplier-products?productId=&supplierId=
  Response: [ SupplierProductDto ]

POST /api/admin/supplier-products
  Request:  { productId, supplierId, cost, supplierSku? }
  Response: SupplierProductDto (201)

PUT /api/admin/supplier-products/{id}
  Request:  { cost, supplierSku? }
  Response: SupplierProductDto
```

### Reportes (admin)

```
GET /api/admin/reports/dashboard
  Response: {
              todaySales: number,
              onlineSales: number,
              posSales: number,
              pendingOrders: number,
              lowStockProducts: number,
              expiringLots: number,
              topProducts: [ { productId, name, quantity } ]
            }

GET /api/admin/reports/cash-session/{id}
  Response: {
              session: CashSessionDto,
              totalsByMethod: {
                CASH: number,
                QR: number,
                TRANSFER: number,
                DEBIT_CARD: number,
                CREDIT_CARD: number
              },
              expectedCash: number,
              countedCash: number,
              difference: number,
              differenceReason: string
            }

GET /api/admin/recommendations
  Response: [
              { type: "LOW_STOCK" | "EXPIRING_SOON" | "HIGH_ROTATION" | "NO_MOVEMENT",
                productId, productName, message, urgency }
            ]
```

### Sucursales (admin)

```
GET /api/admin/branches
  Response: [ BranchDto ]

POST /api/admin/branches
  Request:  { name, address?, phone? }
  Response: BranchDto (201)

PUT /api/admin/branches/{id}
  Request:  { name?, address?, phone? }
  Response: BranchDto
```

---

## Rate limiting

| Endpoint | Limite | Motivo |
|---|---|---|
| `POST /api/auth/login` | 5/min por IP | Fuerza bruta |
| `POST /api/auth/register` | 5/min por IP | Creacion masiva |
| `GET /api/store/products` | 60/min por IP | Catalogo |
| `/api/webhooks/mercadopago` | Sin limite (verificar firma) | MP |

---

> [!seealso] Notas relacionadas
> - [[Backend]]
> - [[Base de Datos]]
> - [[Testing]]
> - Volver a [[_Index]]
