---
title: "Diseno de Base de Datos"
tags:
  - arquitectura
  - base-de-datos
  - postgresql
  - esquema
  - arquitectura-detallada
---

# Diseno de Base de Datos

> [!info] Motor, esquema, patrones de diseno, migraciones y estrategia de concurrencia.

---

## Motor y justificacion

**PostgreSQL 16** es la eleccion por las siguientes razones arquitectonicas:

| Aspecto | Por que PostgreSQL |
|---|---|
| Transaccionalidad | ACID real para operaciones criticas (stock, ventas, pedidos) |
| Bloqueo granular | `SELECT ... FOR UPDATE` para evitar condiciones de carrera en stock |
| Indices avanzados | Parciales, compuestos, GIN para busqueda textual de productos |
| JSON nativo | Util para flexibilidad futura (metadatos de producto, configuraciones) |
| Migraciones versionadas | Flyway se integra naturalmente con el schema |
| Sin licenciamiento | Ideal para tesis academica y despliegue en produccion real |

---

## Arquitectura de esquema

### Principios de diseno

1. **Separacion global vs sucursal**: el catalogo (productos, categorias, marcas, etiquetas) es global a la empresa; el stock, ventas y preparacion pertenecen a sucursales especificas.
2. **Dos fuentes de verdad evitadas**: el precio vigente vive en `product_prices`; el historial en `price_history`. Nunca se duplica la informacion actual en dos tablas.
3. **Trazabilidad no destructiva**: los movimientos de stock no se borran ni se modifican; se anulan con movimientos inversos. Auditoria de acciones criticas en `audit_logs`.
4. **Reserva como entidad de primer orden**: la reserva de stock (`stock_reservations`) es una entidad con ciclo de vida propio, no un simple flag. Esto permite trazabilidad.
5. **Transacciones explicitas en operaciones criticas**: toda operacion que involucre stock + venta/pedido + pago debe ocurrir dentro de una misma transaccion de base de datos.
6. **Promociones orientadas a targeting flexible**: `promotion_targets` permite apuntar a producto, categoria o sucursal, en lugar de una relacion rigida producto-sucursal. Las promociones por lote individual quedan fuera del MVP.

---

## Estrategia de migraciones (Flyway)

### Principios

1. **Naming convention**: `V{version}__{descripcion_corta_con_underscores}.sql`
2. **Una migracion por cambio logico**: no mezclar cambios no relacionados.
3. **Migraciones irreversibles**: no se escribe `DOWN` (se corrige hacia adelante con otra migracion).
4. **Seed data en migraciones separadas**: con `R__` (repeatable) para datos de referencia.

### Migraciones iniciales propuestas

```text
V1__core.sql                        -- companies, branches, users, roles, user_roles
V2__catalog.sql                     -- categories, brands, tags, products, product_tags, product_images
V3__pricing.sql                     -- product_prices, price_history
V4__inventory.sql                   -- branch_stock, stock_lots, stock_movements, stock_reservations
V5__promotions.sql                  -- promotions, promotion_targets
V6__suppliers.sql                   -- suppliers, supplier_products
V7__sales.sql                       -- sales, sale_items
V8__orders.sql                      -- online_orders, online_order_items, payments, deliveries
V9__labels.sql                      -- label_print_jobs, label_print_job_items
V10__assistant.sql                  -- assistant_queries
V11__audit.sql                      -- audit_logs
V12__seed_reference_data.sql        -- roles, metodos de pago, estados
V13__seed_demo_data.sql             -- datos demo para desarrollo
```

---

## Estrategia de concurrencia y transacciones

### El problema critico

Dos clientes compran simultaneamente el ultimo producto disponible. Sin control de concurrencia, ambos pedidos se confirmarian y el stock quedaria negativo.

### Solucion: Bloqueo pesimista con `SELECT ... FOR UPDATE`

```text
BEGIN;
  SELECT physical_stock, reserved_stock
  FROM branch_stock
  WHERE product_id = ? AND branch_id = ?
  FOR UPDATE;  -- Bloquea la fila hasta que termine la transaccion

  -- Validar que (physical_stock - reserved_stock) >= cantidad solicitada
  -- INSERT reserva o descontar stock
  -- UPDATE branch_stock
COMMIT;
```

**Por que pesimista y no optimista**: en una dietetica, las colisiones de stock son poco frecuentes pero cuando ocurren son criticas (el cliente ve "disponible", compra, y recibe "no hay stock"). El bloqueo pesimista garantiza que nunca se confirme una venta sin stock real. Para el volumen de ventas de una dietetica (decenas, no miles de transacciones concurrentes), el overhead del bloqueo es despreciable.

### Niveles de aislamiento recomendados

| Operacion | Aislamiento | Justificacion |
|---|---|---|
| Consulta de catalogo | READ COMMITTED | Lecturas sucias no son problema |
| Venta presencial | READ COMMITTED + FOR UPDATE | Una venta por vez por empleado |
| Crear pedido online | READ COMMITTED | Stock se valida pero no se bloquea |
| Confirmar pago + reservar | REPEATABLE READ | Garantiza que los datos leidos no cambien |
| Cancelar pedido + liberar | READ COMMITTED + FOR UPDATE | Similar a venta |
| Importacion de lista precios | READ COMMITTED | Operacion larga, sin contención |

---

## Indices y rendimiento

| Tabla | Indice | Tipo | Justificacion |
|---|---|---|---|
| `branch_stock` | `(product_id, branch_id)` | UNIQUE | Consulta principal de stock |
| `stock_movements` | `(product_id, created_at)` | Compuesto | Historial cronologico por producto |
| `stock_lots` | `(product_id, branch_id, expiration_date)` | Compuesto | FEFO: lotes ordenados por vencimiento |
| `products` | `(company_id, barcode)` | UNIQUE parcial | Busqueda por codigo de barras |
| `products` | `(company_id, online_status, name)` | Compuesto parcial | Catalogo publico |
| `online_orders` | `(company_id, status, created_at)` | Compuesto parcial | Pedidos activos ordenados por fecha |
| `audit_logs` | `(company_id, entity_type, entity_id)` | Compuesto | Auditoria por entidad |
| `stock_reservations` | `(order_item_id)` | Unico | Reserva por item de pedido |

### Indices parciales (PostgreSQL)

```sql
-- Solo productos publicados (el 99% de las busquedas del catalogo)
CREATE INDEX idx_products_published
  ON products(company_id, name)
  WHERE online_status = 'PUBLISHED';

-- Solo pedidos activos (excluye ENTREGADO y CANCELADO)
CREATE INDEX idx_orders_active
  ON online_orders(company_id, created_at)
  WHERE status NOT IN ('ENTREGADO', 'CANCELADO');

-- Solo reservas activas
CREATE INDEX idx_reservations_active
  ON stock_reservations(created_at)
  WHERE status = 'ACTIVA';
```

---

## Estrategia de auditoria

### Que auditar

| Accion | Datos registrados |
|---|---|
| Cambio de precio | valor_anterior, valor_nuevo, usuario, fecha |
| Ajuste de stock | producto, sucursal, cantidad, motivo, usuario |
| Cancelacion de pedido | pedido_id, estado_anterior, motivo, usuario |
| Cambio de estado de pago | pago_id, estado_anterior, estado_nuevo |
| Creacion/modificacion de usuario | usuario_afectado, cambios, quien_lo_hizo |
| Cambio de rol | usuario_afectado, rol_anterior, rol_nuevo |
| Importacion de lista precios | proveedor, resultado, items detectados |

La auditoria se implementa de forma **programatica** desde los servicios (no triggers en BD). Esto da control total sobre que se audita y permite incluir metadata de negocio (como el motivo) que un trigger no puede capturar.

---

## Seed data

### Datos fijos de referencia

- Roles: `ADMIN_GENERAL`, `ENCARGADO_SUCURSAL`, `EMPLEADO`, `CLIENTE`
- Metodos de pago: `CASH`, `TRANSFER`, `QR`, `LINK_PAGO` (pago online simulado)
- Tipos de movimiento: `INGRESO`, `VENTA`, `AJUSTE`, `MERMA`, `CONSUMO_INTERNO`, `RESERVA`, `CANCELACION`

### Datos demo

- 2 sucursales: "Centro" y "Nueva Cordoba"
- 4 categorias, 15-20 productos, stock inicial en ambas sucursales
- 1 proveedor demo, 1 admin, 1 empleado

---

## Diagrama completo del esquema

> Organizado por dominios: Core, Catalogo, Inventario, Promociones, Proveedores, Ventas, Pedidos, Impresion, IA y Auditoria.

```mermaid
%%{init: {
  "theme": "base",
  "themeVariables": {
    "darkMode": true,
  }
}}%%
erDiagram

%% ==================== CORE / EMPRESA ====================

    companies {
        bigint id PK
        varchar name
        varchar legal_name
        varchar cuit UK
        varchar email
        varchar phone
        timestamp created_at
    }

    branches {
        bigint id PK
        bigint company_id FK
        varchar name
        varchar address
        varchar phone
        varchar manager_name
        boolean active
        timestamp created_at
    }

    users {
        bigint id PK
        bigint company_id FK
        bigint default_branch_id FK "nullable"
        varchar email UK
        varchar password_hash
        varchar name
        varchar phone
        boolean active
        timestamp created_at
    }

    roles {
        varchar code PK "ADMIN_GENERAL|ENCARGADO_SUCURSAL|EMPLEADO|CLIENTE"
        varchar name
        varchar description
    }

    user_roles {
        bigint user_id PK,FK
        varchar role_code PK,FK
    }

%% ==================== CATALOGO ====================

    categories {
        bigint id PK
        bigint company_id FK
        bigint parent_id FK "nullable, auto-referencia"
        varchar name
        varchar description
    }

    brands {
        bigint id PK
        bigint company_id FK
        varchar name
        varchar description
    }

    tags {
        bigint id PK
        bigint company_id FK
        varchar code "sin-tacc|vegano|sin-azucar"
        varchar name
    }

    products {
        bigint id PK
        bigint company_id FK
        bigint category_id FK
        bigint brand_id FK "nullable"
        varchar name
        text description
        varchar barcode UK "nullable"
        varchar sku UK
        varchar online_status "DRAFT|PUBLISHED|PAUSED|DISABLED" "reemplaza a published_online + active para visibilidad online"
        boolean requires_expiration
        varchar unit "unidad|kg|g|lt"
        boolean active "indica si el producto esta activo en el sistema (no necesariamente publicado)"
        timestamp created_at
        timestamp updated_at
    }

    product_tags {
        bigint product_id PK,FK
        bigint tag_id PK,FK
    }

    product_images {
        bigint id PK
        bigint product_id FK
        varchar file_path
        int sort_order
        boolean is_main
        timestamp uploaded_at
    }

    product_prices {
        bigint product_id PK,FK
        decimal base_price
        decimal cost_price "nullable"
        decimal profit_margin_percentage "nullable"
        bigint updated_by FK
        timestamp updated_at
    }

    price_history {
        bigint id PK
        bigint product_id FK
        decimal old_price
        decimal new_price
        decimal old_cost "nullable"
        decimal new_cost "nullable"
        varchar reason
        bigint changed_by FK
        timestamp created_at
    }

%% ==================== INVENTARIO / STOCK ====================

    branch_stock {
        bigint product_id PK,FK
        bigint branch_id PK,FK
        int physical_stock
        int reserved_stock
        int minimum_stock "nullable"
        timestamp updated_at
    }

    stock_lots {
        bigint id PK
        bigint product_id FK
        bigint branch_id FK
        varchar lot_number "nullable"
        int quantity_available
        date expiration_date "nullable"
        timestamp received_at
        timestamp created_at
    }

    stock_reservations {
        bigint id PK
        bigint order_item_id FK
        bigint product_id FK
        bigint branch_id FK
        bigint stock_lot_id FK "nullable"
        int quantity
        varchar status "ACTIVA|CONFIRMADA|LIBERADA"
        timestamp expires_at "nullable, solo aplica si en futuro se reserva antes del pago"
        timestamp created_at
        timestamp updated_at
    }

    stock_movements {
        bigint id PK
        bigint product_id FK
        bigint branch_id FK
        bigint stock_lot_id FK "nullable"
        bigint user_id FK
        varchar movement_type "INGRESO|VENTA|AJUSTE|MERMA|CONSUMO_INTERNO|RESERVA|CANCELACION"
        int quantity_delta "positivo o negativo"
        varchar reference_type "sale|online_order|adjustment|supplier_import"
        bigint reference_id "nullable"
        text reason "nullable"
        timestamp created_at
    }

%% ==================== PROMOCIONES ====================

    promotions {
        bigint id PK
        bigint company_id FK
        varchar name
        text description
        varchar discount_type "PERCENTAGE|FIXED_AMOUNT|FIXED_PRICE"
        decimal discount_value
        timestamp start_date
        timestamp end_date
        boolean active
        bigint created_by FK
        timestamp created_at
    }

    promotion_targets {
        bigint id PK
        bigint promotion_id FK
        bigint product_id FK "nullable"
        bigint category_id FK "nullable"
        bigint branch_id FK "nullable"
    }

%% ==================== PROVEEDORES ====================

    suppliers {
        bigint id PK
        bigint company_id FK
        varchar name
        varchar contact_name
        varchar phone
        varchar email
        varchar cuit
        varchar address
        timestamp created_at
    }

    supplier_products {
        bigint id PK
        bigint product_id FK
        bigint supplier_id FK
        varchar supplier_sku "nullable"
        decimal current_cost
        boolean is_preferred
        timestamp updated_at
    }

    supplier_cost_history {
        bigint id PK
        bigint supplier_product_id FK
        decimal old_cost
        decimal new_cost
        varchar reason "nullable"
        bigint changed_by FK
        timestamp created_at
    }

%% ==================== VENTAS PRESENCIALES ====================

    sales {
        bigint id PK
        bigint branch_id FK
        bigint user_id FK
        bigint customer_user_id FK "nullable"
        decimal subtotal_amount
        decimal discount_amount
        decimal total_amount
        varchar payment_method "CASH|TRANSFER|QR"
        timestamp created_at
    }

    sale_items {
        bigint id PK
        bigint sale_id FK
        bigint product_id FK
        bigint stock_lot_id FK "nullable"
        int quantity
        decimal unit_price
        decimal discount_amount
        varchar product_name_snapshot "snapshot al momento de la venta"
        varchar sku_snapshot "snapshot al momento de la venta"
        decimal cost_price_snapshot "costo al momento de la venta"
    }

%% ==================== PEDIDOS ONLINE ====================

    online_orders {
        bigint id PK
        bigint company_id FK
        bigint branch_id FK "sucursal que prepara/reserva"
        bigint customer_user_id FK "nullable"
        varchar order_number UK
        varchar customer_name
        varchar customer_email "nullable"
        varchar customer_phone
        varchar status "PENDIENTE_PAGO|PAGADO|EN_PREPARACION|LISTO_PARA_RETIRAR|EN_REPARTO|ENTREGADO|CANCELADO"
        varchar delivery_type "PICKUP|SHIPPING"
        decimal subtotal_amount
        decimal discount_amount
        decimal total_amount
        timestamp created_at
        timestamp updated_at
    }

    online_order_items {
        bigint id PK
        bigint order_id FK
        bigint product_id FK
        int quantity
        decimal unit_price
        decimal discount_amount
        varchar product_name_snapshot "snapshot al momento del pedido"
        varchar sku_snapshot "snapshot al momento del pedido"
        decimal cost_price_snapshot "costo al momento del pedido"
    }

    payments {
        bigint id PK
        bigint order_id FK
        varchar status "PENDIENTE|APROBADO|RECHAZADO|CANCELADO"
        decimal amount
        varchar payment_method "LINK_PAGO|QR"
        varchar external_reference "nullable"
        varchar payment_link "nullable"
        timestamp created_at
        timestamp confirmed_at "nullable"
    }

    deliveries {
        bigint id PK
        bigint order_id FK
        varchar type "PICKUP|SHIPPING"
        varchar address "nullable"
        varchar city "nullable"
        varchar phone "nullable"
        text notes "nullable"
        bigint prepared_by FK "nullable"
        timestamp prepared_at "nullable"
        timestamp delivered_at "nullable"
    }

%% ==================== IMPRESION ====================

    label_print_jobs {
        bigint id PK
        bigint branch_id FK "nullable"
        bigint created_by FK
        varchar job_type "PRICE_TAG|INTERNAL|PROMOTION"
        varchar output_path "nullable"
        timestamp created_at
    }

    label_print_job_items {
        bigint id PK
        bigint job_id FK
        bigint product_id FK
        int quantity
    }

%% ==================== IA / ASISTENTE ====================

    assistant_queries {
        bigint id PK
        bigint company_id FK
        bigint branch_id FK "nullable"
        bigint user_id FK "nullable"
        varchar query_type "CUSTOMER|ADMIN"
        text query_text
        text response_text
        jsonb recommended_products "nullable"
        timestamp created_at
    }

%% ==================== AUDITORIA ====================

    audit_logs {
        bigint id PK
        bigint company_id FK
        varchar entity_type
        bigint entity_id
        varchar action
        jsonb old_values "nullable"
        jsonb new_values "nullable"
        varchar reason "nullable"
        bigint user_id FK
        timestamp created_at
    }

%% ==================== RELACIONES: CORE ====================

    companies ||--o{ branches : tiene
    companies ||--o{ users : administra
    companies ||--o{ categories : define
    companies ||--o{ brands : registra
    companies ||--o{ tags : define
    companies ||--o{ products : posee
    companies ||--o{ suppliers : trabaja_con
    companies ||--o{ promotions : crea
    companies ||--o{ online_orders : recibe
    companies ||--o{ assistant_queries : registra
    companies ||--o{ audit_logs : audita

    branches ||--o{ users : asigna_default
    branches ||--o{ branch_stock : resume_stock
    branches ||--o{ stock_lots : almacena_lotes
    branches ||--o{ stock_movements : registra_movimientos
    branches ||--o{ sales : registra_ventas
    branches ||--o{ online_orders : prepara_pedidos
    branches ||--o{ promotion_targets : contextualiza_promo
    branches ||--o{ label_print_jobs : imprime_etiquetas
    branches ||--o{ assistant_queries : filtra_consulta

    users ||--o{ user_roles : tiene_roles
    roles ||--o{ user_roles : asignado_a

    users ||--o{ stock_movements : registra
    users ||--o{ sales : realiza
    users ||--o{ product_prices : actualiza_precio
    users ||--o{ price_history : cambia_precio
    users ||--o{ promotions : crea_promocion
    users ||--o{ deliveries : prepara
    users ||--o{ label_print_jobs : genera_trabajo
    users ||--o{ assistant_queries : consulta
    users ||--o{ audit_logs : ejecuta_accion

%% ==================== RELACIONES: CATALOGO ====================

    categories ||--o{ categories : tiene_subcategorias
    categories ||--o{ products : agrupa
    categories ||--o{ promotion_targets : aplica_categoria

    brands ||--o{ products : marca

    tags ||--o{ product_tags : clasifica
    products ||--o{ product_tags : tiene_tags
    products ||--o{ product_images : tiene_imagenes
    products ||--o| product_prices : tiene_precio_vigente
    products ||--o{ price_history : tiene_historial

%% ==================== RELACIONES: INVENTARIO ====================

    products ||--o{ branch_stock : tiene_stock_por_sucursal
    products ||--o{ stock_lots : tiene_lotes
    products ||--o{ stock_movements : mueve_stock
    products ||--o{ stock_reservations : reserva_stock

    branch_stock ||--o{ stock_reservations : contiene_reservas

    stock_lots ||--o{ stock_movements : origina_movimientos
    stock_lots ||--o{ stock_reservations : reserva_lote_especifico
    stock_lots ||--o{ sale_items : descuenta_lote
%% ==================== RELACIONES: PROMOCIONES ====================

    promotions ||--o{ promotion_targets : aplica_a
    promotion_targets }o--|| products : targeting_opcional
    promotion_targets }o--|| categories : targeting_opcional
    promotion_targets }o--|| branches : targeting_opcional
%% ==================== RELACIONES: PROVEEDORES ====================

    suppliers ||--o{ supplier_products : ofrece
    supplier_products ||--o{ supplier_cost_history : registra_cambios
    products ||--o{ supplier_products : abastecido_por

%% ==================== RELACIONES: VENTAS ====================

    sales ||--o{ sale_items : contiene
    products ||--o{ sale_items : vendido_en
    sale_items }o--|| stock_lots : opcional

%% ==================== RELACIONES: PEDIDOS ====================

    online_orders ||--o{ online_order_items : contiene
    products ||--o{ online_order_items : pedido_en
    online_order_items ||--o{ stock_reservations : genera_reservas
    online_orders ||--o{ payments : tiene_intentos_pago
    online_orders ||--o| deliveries : tiene_entrega

%% ==================== RELACIONES: IMPRESION ====================

    label_print_jobs ||--o{ label_print_job_items : contiene
    products ||--o{ label_print_job_items : impreso_en

%% ==================== RELACIONES: IA ====================

    assistant_queries }o--|| branches : contexto_opcional
    assistant_queries }o--|| users : origen_opcional
```

---

## Glosario de tablas

| # | Tabla | Dominio | Proposito |
|---|---|---|---|
| 1 | `companies` | Core | Empresa duena del sistema |
| 2 | `branches` | Core | Sucursales fisicas |
| 3 | `users` | Core | Usuarios del sistema |
| 4 | `roles` | Core | Roles fijos del sistema |
| 5 | `user_roles` | Core | Asignacion usuario-rol |
| 6 | `categories` | Catalogo | Categorias de producto (arbol) |
| 7 | `brands` | Catalogo | Marcas de producto |
| 8 | `tags` | Catalogo | Etiquetas (sin TACC, vegano, etc.) |
| 9 | `products` | Catalogo | Productos del catalogo |
| 10 | `product_tags` | Catalogo | Tags por producto |
| 11 | `product_images` | Catalogo | Imagenes de producto |
| 12 | `product_prices` | Catalogo | Precio vigente del producto |
| 13 | `price_history` | Catalogo | Cambios historicos de precio |
| 14 | `branch_stock` | Inventario | Stock agregado por sucursal |
| 15 | `stock_lots` | Inventario | Lotes con vencimiento |
| 16 | `stock_reservations` | Inventario | Reservas de stock para pedidos |
| 17 | `stock_movements` | Inventario | Trazabilidad de movimientos |
| 18 | `promotions` | Promociones | Cabecera de promocion |
| 19 | `promotion_targets` | Promociones | Targeting de promocion |
| 20 | `suppliers` | Proveedores | Proveedores |
| 21 | `supplier_products` | Proveedores | Producto por proveedor |
| 22 | `supplier_cost_history` | Proveedores | Historial de cambios de costo por proveedor |
| 23 | `sales` | Ventas | Ventas presenciales |
| 24 | `sale_items` | Ventas | Items de venta |
| 25 | `online_orders` | Pedidos | Pedidos online |
| 26 | `online_order_items` | Pedidos | Items de pedido |
| 27 | `payments` | Pedidos | Pagos de pedidos |
| 28 | `deliveries` | Pedidos | Entregas de pedidos |
| 29 | `label_print_jobs` | Impresion | Trabajos de impresion de etiquetas |
| 30 | `label_print_job_items` | Impresion | Items incluidos en el trabajo |
| 31 | `assistant_queries` | IA | Consultas al asistente inteligente |
| 32 | `audit_logs` | Auditoria | Auditoria de acciones criticas |

---

> [!seealso] Notas relacionadas
> - [[Modelo de Datos]] -- entidades y relaciones del dominio
> - [[Backend]] -- capa de persistencia en Spring Boot
> - [[Vision General]] -- stack tecnologico
> - Volver a [[_Index]]
