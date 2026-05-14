---
title: "Diseno de Base de Datos"
tags:
  - arquitectura
  - base-de-datos
  - postgresql
  - esquema
---

# Diseno de Base de Datos

> [!info] Motor: PostgreSQL 16. Esquema completo con 14 tablas para el MVP.

---

## Principios de diseno

1. **Catalogo global, stock por sucursal**: productos y categorias globales. Stock en lotes y ordenes por sucursal.
2. **stock_lots como unica fuente de verdad del stock**: disponible = SUM(quantity_available) de lotes activos. Sin stock_reservations.
3. **Orden unificada**: `orders` unifica venta presencial (POS) y pedido online (ONLINE).
4. **Payments unificados**: tabla `payments` sirve tanto para pagos online (MP) como presenciales (caja).
5. **Caja operativa**: `cash_sessions` representa un turno de caja. Toda venta presencial requiere caja abierta.
6. **Precio en el producto**: `products.sale_price`. Historial en audit_logs.
7. **Rol como campo directo**: `users.role`. Sin tablas roles ni user_roles.
8. **Stock descontado al aprobar pago**: no hay reservas. Se descuenta cuando el pago se aprueba.
9. **Snapshots en items de orden**: `order_items` guarda datos del producto al momento de la venta.

---

## Migraciones Flyway

```text
V1__core.sql                        -- branches, users
V2__catalog.sql                     -- categories, products
V3__suppliers.sql                   -- suppliers, supplier_products
V4__inventory.sql                   -- stock_lots, stock_movements
V5__orders.sql                      -- orders, order_items
V6__payments.sql                    -- payments
V7__cash.sql                        -- cash_sessions, cash_movements
V8__optional_promotions.sql         -- product_promotions
V9__audit.sql                       -- audit_logs
V10__seed_data.sql                  -- datos demo
```

---

## Transacciones y concurrencia

Toda operacion que cruza stock + orden + pago debe ser transaccional.

### Estrategia: bloqueo pesimista con SELECT...FOR UPDATE

```text
BEGIN;
  SELECT id, quantity_available
  FROM stock_lots
  WHERE product_id = ? AND branch_id = ? AND quantity_available > 0
  ORDER BY expiration_date ASC
  FOR UPDATE;

  -- Validar SUM(quantity_available) >= cantidad solicitada
  -- UPDATE stock_lots (descontar FEFO)
  -- INSERT stock_movement
  -- INSERT/UPDATE order
  -- INSERT/UPDATE payment
COMMIT;
```

### Operaciones transaccionales

| Operacion | Tablas | Estrategia |
|---|---|---|
| Venta presencial (POS) | orders, order_items, stock_lots, stock_movements, payments, cash_sessions | FOR UPDATE |
| Confirmar pago MP (webhook) | orders, payments, stock_lots, stock_movements | FOR UPDATE |
| Cancelar orden | orders, stock_lots, stock_movements, payments | FOR UPDATE |
| Registrar ingreso de stock | stock_lots, stock_movements | READ COMMITTED |

---

## Tablas del esquema

### Core

```sql
CREATE TABLE branches (
    id BIGSERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    address VARCHAR(255),
    phone VARCHAR(50),
    active BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE users (
    id BIGSERIAL PRIMARY KEY,
    branch_id BIGINT REFERENCES branches(id),
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    phone VARCHAR(50),
    role VARCHAR(20) NOT NULL CHECK (role IN ('ADMIN','MANAGER','EMPLOYEE','CUSTOMER')),
    enabled BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);
```

### Catalogo

```sql
CREATE TABLE categories (
    id BIGSERIAL PRIMARY KEY,
    parent_id BIGINT REFERENCES categories(id),
    name VARCHAR(255) NOT NULL,
    description TEXT
);

CREATE TABLE products (
    id BIGSERIAL PRIMARY KEY,
    category_id BIGINT REFERENCES categories(id),
    name VARCHAR(255) NOT NULL,
    description TEXT,
    brand_name VARCHAR(255),
    barcode VARCHAR(100) UNIQUE,
    online_status VARCHAR(20) DEFAULT 'DRAFT' CHECK (online_status IN ('DRAFT','PUBLISHED','PAUSED','HIDDEN')),
    image_url VARCHAR(500),
    sale_price DECIMAL(12,2) NOT NULL CHECK (sale_price >= 0),
    minimum_stock INT,
    active BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);
```

### Proveedores

```sql
CREATE TABLE suppliers (
    id BIGSERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    contact_name VARCHAR(255),
    phone VARCHAR(50),
    email VARCHAR(255),
    cuit VARCHAR(20) UNIQUE,
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE supplier_products (
    id BIGSERIAL PRIMARY KEY,
    product_id BIGINT REFERENCES products(id),
    supplier_id BIGINT REFERENCES suppliers(id),
    supplier_sku VARCHAR(100),
    current_cost DECIMAL(12,2) NOT NULL CHECK (current_cost >= 0),
    is_preferred BOOLEAN DEFAULT false,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);
```

### Inventario

```sql
CREATE TABLE stock_lots (
    id BIGSERIAL PRIMARY KEY,
    product_id BIGINT REFERENCES products(id),
    branch_id BIGINT REFERENCES branches(id),
    lot_code VARCHAR(100),
    expiration_date DATE,
    quantity_available DECIMAL(12,3) NOT NULL CHECK (quantity_available >= 0),
    cost_price DECIMAL(12,2) CHECK (cost_price >= 0),
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE stock_movements (
    id BIGSERIAL PRIMARY KEY,
    product_id BIGINT REFERENCES products(id),
    branch_id BIGINT REFERENCES branches(id),
    stock_lot_id BIGINT REFERENCES stock_lots(id),
    type VARCHAR(50) NOT NULL CHECK (type IN ('PURCHASE_ENTRY','POS_SALE','ONLINE_SALE','CANCELLATION_RETURN','MANUAL_ADJUSTMENT','WASTE','INTERNAL_CONSUMPTION')),
    quantity DECIMAL(12,3) NOT NULL,
    reason TEXT,
    order_id BIGINT,
    created_by_user_id BIGINT REFERENCES users(id),
    created_at TIMESTAMP DEFAULT NOW()
);
```

### Ordenes

```sql
CREATE TABLE orders (
    id BIGSERIAL PRIMARY KEY,
    order_number VARCHAR(50) UNIQUE NOT NULL,
    type VARCHAR(20) NOT NULL CHECK (type IN ('POS','ONLINE')),
    status VARCHAR(30) NOT NULL CHECK (status IN ('PENDING_PAYMENT','PAID','PREPARING','READY','DELIVERED','CANCELLED','PAYMENT_FAILED','STOCK_CONFLICT')),
    branch_id BIGINT REFERENCES branches(id),
    customer_user_id BIGINT REFERENCES users(id),
    created_by_user_id BIGINT REFERENCES users(id),
    customer_name_snapshot VARCHAR(255),
    customer_email_snapshot VARCHAR(255),
    customer_phone_snapshot VARCHAR(50),
    fulfillment_type VARCHAR(20) DEFAULT 'PICKUP',
    subtotal DECIMAL(12,2) NOT NULL CHECK (subtotal >= 0),
    discount_total DECIMAL(12,2) DEFAULT 0 CHECK (discount_total >= 0),
    total DECIMAL(12,2) NOT NULL CHECK (total >= 0),
    notes TEXT,
    paid_at TIMESTAMP,
    prepared_at TIMESTAMP,
    delivered_at TIMESTAMP,
    cancelled_at TIMESTAMP,
    cancellation_reason TEXT,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE order_items (
    id BIGSERIAL PRIMARY KEY,
    order_id BIGINT REFERENCES orders(id),
    product_id BIGINT REFERENCES products(id),
    quantity DECIMAL(12,3) NOT NULL CHECK (quantity > 0),
    unit_price DECIMAL(12,2) NOT NULL CHECK (unit_price >= 0),
    discount_amount DECIMAL(12,2) DEFAULT 0,
    subtotal_amount DECIMAL(12,2) NOT NULL CHECK (subtotal_amount >= 0),
    product_name_snapshot VARCHAR(255) NOT NULL,
    product_barcode_snapshot VARCHAR(100),
    cost_price_snapshot DECIMAL(12,2),
    created_at TIMESTAMP DEFAULT NOW()
);
```

### Pagos (unificados)

```sql
CREATE TABLE payments (
    id BIGSERIAL PRIMARY KEY,
    order_id BIGINT REFERENCES orders(id) NOT NULL,
    cash_session_id BIGINT REFERENCES cash_sessions(id),
    provider VARCHAR(50) NOT NULL CHECK (provider IN ('MERCADO_PAGO','MANUAL','BANK','CARD_TERMINAL')),
    method VARCHAR(50) NOT NULL CHECK (method IN ('CHECKOUT_PRO','CASH','QR','TRANSFER','DEBIT_CARD','CREDIT_CARD','OTHER')),
    status VARCHAR(20) NOT NULL CHECK (status IN ('PENDING','APPROVED','REJECTED','CANCELLED','REFUNDED','EXPIRED','IN_PROCESS')),
    amount DECIMAL(12,2) NOT NULL CHECK (amount > 0),
    currency VARCHAR(3) DEFAULT 'ARS',
    provider_payment_id VARCHAR(255),
    provider_preference_id VARCHAR(255),
    external_reference VARCHAR(255),
    approved_at TIMESTAMP,
    metadata JSONB,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);
```

### Caja

```sql
CREATE TABLE cash_sessions (
    id BIGSERIAL PRIMARY KEY,
    branch_id BIGINT REFERENCES branches(id) NOT NULL,
    opened_by_user_id BIGINT REFERENCES users(id) NOT NULL,
    closed_by_user_id BIGINT REFERENCES users(id),
    opened_at TIMESTAMP DEFAULT NOW(),
    closed_at TIMESTAMP,
    opening_cash_amount DECIMAL(12,2) NOT NULL CHECK (opening_cash_amount >= 0),
    expected_cash_amount DECIMAL(12,2),
    counted_cash_amount DECIMAL(12,2),
    cash_difference_amount DECIMAL(12,2),
    cash_difference_reason TEXT,
    status VARCHAR(10) NOT NULL DEFAULT 'OPEN' CHECK (status IN ('OPEN','CLOSED')),
    opening_notes TEXT,
    closing_notes TEXT,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE cash_movements (
    id BIGSERIAL PRIMARY KEY,
    cash_session_id BIGINT REFERENCES cash_sessions(id) NOT NULL,
    created_by_user_id BIGINT REFERENCES users(id) NOT NULL,
    type VARCHAR(20) NOT NULL CHECK (type IN ('CASH_IN','CASH_OUT','ADJUSTMENT')),
    method VARCHAR(20) NOT NULL CHECK (method IN ('CASH','TRANSFER','OTHER')),
    amount DECIMAL(12,2) NOT NULL,
    reason TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT NOW()
);
```

### Auditoria

```sql
CREATE TABLE audit_logs (
    id BIGSERIAL PRIMARY KEY,
    user_id BIGINT REFERENCES users(id),
    action VARCHAR(100) NOT NULL,
    entity_type VARCHAR(100) NOT NULL,
    entity_id BIGINT,
    description TEXT,
    created_at TIMESTAMP DEFAULT NOW()
);
```

---

## Constraints principales

| Tabla | Constraint | Justificacion |
|---|---|---|
| `products` | `barcode UNIQUE` | Codigo de barras unico |
| `products` | `sale_price >= 0` | Precio no negativo |
| `stock_lots` | `quantity_available >= 0` | Stock nunca negativo |
| `order_items` | `quantity > 0, unit_price >= 0, subtotal_amount >= 0` | Consistencia |
| `orders` | `total >= 0, order_number UNIQUE` | Consistencia |
| `payments` | `amount > 0` | Importe positivo |
| `cash_sessions` | Solo una OPEN por branch (logica de aplicacion) | Integridad de caja |
| `cash_movements` | `amount != 0` (logica de aplicacion) | Movimiento no nulo |

## Indices principales

| Tabla | Indice | Justificacion |
|---|---|---|
| `stock_lots` | `(product_id, branch_id, expiration_date)` | FEFO |
| `stock_lots` | `(expiration_date)` | Alertas vencimiento |
| `stock_movements` | `(product_id, created_at)` | Historial por producto |
| `stock_movements` | `(order_id)` | Movimientos por orden |
| `products` | `(barcode)` | Busqueda por codigo |
| `products` | `(category_id)` | Filtro |
| `products` | `(online_status)` | Catalogo publico |
| `orders` | `(branch_id, created_at)` | Ordenes por sucursal |
| `orders` | `(status)` | Filtro por estado |
| `orders` | `(type)` | Filtro POS/ONLINE |
| `payments` | `(order_id)` | Pagos por orden |
| `payments` | `(cash_session_id)` | Pagos por caja |
| `payments` | `(provider_payment_id)` | Busqueda MP |
| `cash_sessions` | `(branch_id, status)` | Caja activa por sucursal |
| `audit_logs` | `(entity_type, entity_id)` | Auditoria por entidad |

---

## Seed data

```text
- 1 sucursal: "Centro"
- Roles: ADMIN, MANAGER, EMPLOYEE, CUSTOMER
- 1 admin demo, 1 customer demo
- 4 categorias, 15-20 productos con stock inicial
- 1 proveedor demo
- Metodos de pago: CHECKOUT_PRO, CASH, QR, TRANSFER, DEBIT_CARD, CREDIT_CARD
- Estados de orden: PENDING_PAYMENT, PAID, PREPARING, READY, DELIVERED, CANCELLED, PAYMENT_FAILED, STOCK_CONFLICT
- Estados de pago: PENDING, APPROVED, REJECTED, CANCELLED, REFUNDED, EXPIRED, IN_PROCESS
- Proveedores de pago: MERCADO_PAGO, MANUAL, BANK, CARD_TERMINAL
```

---

> [!seealso] Notas relacionadas
> - [[Modelo de Datos]]
> - [[Backend]]
> - [[Endpoints API]]
> - Volver a [[_Index]]
