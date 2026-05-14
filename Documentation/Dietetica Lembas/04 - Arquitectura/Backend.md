---
title: "Diseno Backend"
tags:
  - arquitectura
  - backend
  - spring-boot
---

# Diseno Backend (MVP)

> [!info] Monolito modular Spring Boot con payments unificados, Mercado Pago y caja operativa.

---

## Principios

1. **Monolito modular**: cada modulo funcional agrupa sus propias clases.
2. **Estructura simple**: entity -> repository -> service -> controller.
3. **Logica compleja extraida localmente**: reglas FEFO, calculo de caja, integracion MP.
4. **Adaptador para MP**: `MercadoPagoGateway` detras de interfaz `PaymentGateway`.
5. **Transaccionalidad explicita**: operaciones que cruzan multiples tablas en una transaccion.

---

## Estructura por modulos

```text
backend/
  auth/             -- Autenticacion JWT, registro, login
  users/            -- CRUD de usuarios internos
  catalog/          -- Productos, categorias
  inventory/        -- StockLot, StockMovement, FEFO
  orders/           -- Ordenes unificadas (POS y ONLINE)
  payments/         -- Payment, MercadoPagoGateway, webhook
  cash/             -- CashSession, CashMovement
  suppliers/        -- Proveedores, supplier_products
  reports/          -- Dashboard, reporte de caja, recomendaciones
  audit/            -- AuditLog
  webhooks/         -- Endpoints de webhook (Mercado Pago)
  shared/           -- DTOs comunes, excepciones, utilidades
```

### Ejemplo: modulo payments

```text
payments/
  model/
    Payment.java
    PaymentProvider.java       -- Enum: MERCADO_PAGO, MANUAL, BANK, CARD_TERMINAL
    PaymentMethod.java         -- Enum: CHECKOUT_PRO, CASH, QR, TRANSFER, etc.
    PaymentStatus.java         -- Enum: PENDING, APPROVED, REJECTED, etc.
  gateway/
    PaymentGateway.java        -- Interface
    MercadoPagoGateway.java    -- Implementacion real con API MP
  service/
    PaymentService.java        -- Creacion, actualizacion, consulta
    MercadoPagoService.java    -- Crear preferencia, verificar pago, procesar webhook
  web/
    PaymentController.java     -- (pocos endpoints, pagos se crean desde orders)
    MercadoPagoWebhook.java    -- POST /api/webhooks/mercadopago
```

### Ejemplo: modulo cash

```text
cash/
  model/
    CashSession.java
    CashMovement.java
    CashMovementType.java
  policy/
    CashCloseCalculator.java   -- Calculo de expectedCashAmount, diferencia
  service/
    CashService.java           -- Abrir, cerrar, movimientos, reporte
  web/
    CashController.java        -- Endpoints de caja
```

### Ejemplo: modulo inventory (FEFO)

```text
inventory/
  model/
    StockLot.java
    StockMovement.java
    StockMovementType.java
  policy/
    FefoStockDeductionPolicy.java   -- Logica FEFO pura, testeable sin Spring
  service/
    InventoryService.java           -- Orquesta Policy + Repository + @Transactional
  web/
    InventoryController.java
```

---

## Mapa de dependencias

```text
auth       -- Sin dependencias
catalog    -- auth
inventory  -- catalog, auth
orders     -- catalog, inventory, payments, auth
payments   -- orders, cash, auth
cash       -- auth
suppliers  -- catalog, auth
reports    -- catalog, inventory, orders, payments, cash, auth
webhooks   -- payments, orders, inventory, auth
audit      -- auth
```

---

## Integracion Mercado Pago

### Flujo

```text
1. Cliente crea orden (POST /api/customer/orders)
2. Cliente solicita checkout (POST /api/customer/orders/{id}/checkout/mp)
3. Backend crea preferencia en MP via MercadoPagoGateway
4. Backend guarda provider_preference_id en payment
5. Backend devuelve init_point al frontend
6. Frontend redirige a MP Checkout Pro
7. Cliente paga en MP
8. MP redirige al frontend (resultado)
9. MP envia webhook a POST /api/webhooks/mercadopago
10. Backend verifica firma y consulta estado real en MP
11. Si APPROVED: actualiza payment, descuenta stock FEFO, order a PAID
12. Idempotencia: mismo provider_payment_id no se procesa dos veces
```

### Adaptador

```java
interface PaymentGateway {
    Preference createPreference(Order order, Payment payment);
    PaymentStatus checkPayment(String paymentId);
    boolean verifyWebhookSignature(WebhookRequest request);
}
```

---

## Arquitectura de seguridad

### Cadena de filtros Spring Security

```text
SecurityFilterChain
  ├── CorsFilter
  ├── CsrfFilter (DISABLED) -- API REST stateless
  ├── JwtAuthenticationFilter
  └── ExceptionHandlerFilter

Rutas publicas: /api/auth/**, /api/store/**, /api/webhooks/**, /uploads/**
Rutas customer: /api/customer/** (rol CUSTOMER)
Rutas admin: /api/admin/** (ADMIN, MANAGER, EMPLOYEE)
```

### PreAuthorize

```text
// Publico
@PermitAll en /api/store/**, /api/auth/**, /api/webhooks/**

// Cliente
@PreAuthorize("hasRole('CUSTOMER')") en /api/customer/**

// Admin
@PreAuthorize("hasAnyRole('ADMIN', 'MANAGER', 'EMPLOYEE')") en /api/admin/**

// Solo ADMIN
@PreAuthorize("hasRole('ADMIN')") en usuarios, configuracion
```

---

## Transacciones

| Operacion | Tablas | Estrategia |
|---|---|---|
| POS sale | orders, order_items, stock_lots, stock_movements, payments, cash_sessions | FOR UPDATE |
| Webhook MP (approve) | payments, orders, stock_lots, stock_movements | FOR UPDATE |
| Cancel order | orders, stock_lots, stock_movements, payments | FOR UPDATE |
| Stock entry | stock_lots, stock_movements | READ COMMITTED |

---

## Manejo de errores

| Modulo | Excepciones |
|---|---|
| `catalog` | `ProductNotFoundException`, `ProductNotPublishedException` |
| `inventory` | `InsufficientStockException`, `LotExpiredException` |
| `orders` | `OrderNotFoundException`, `OrderInvalidStateException` |
| `payments` | `PaymentFailedException`, `MercadoPagoException` |
| `cash` | `CashSessionNotOpenException`, `CashSessionAlreadyOpenException`, `CashDifferenceException` |
| `auth` | `InvalidCredentialsException`, `AccountDisabledException` |

Todas heredan de `DomainException` y se manejan con `@ControllerAdvice`.

---

> [!seealso] Notas relacionadas
> - [[Base de Datos]]
> - [[Endpoints API]]
> - Volver a [[_Index]]
