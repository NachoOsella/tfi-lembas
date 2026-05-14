---
title: "Diseno Frontend"
tags:
  - arquitectura
  - frontend
  - angular
---

# Diseno Frontend (MVP)

> [!info] Angular con Signals, carrito local, integracion con Mercado Pago Checkout Pro, y modulos de caja.

---

## Principios

1. **Componentes sin logica de negocio**: la logica vive en servicios.
2. **Estado simple**: Signals de Angular. Sin NgRx.
3. **Lazy loading**: backoffice se carga solo cuando es necesario.
4. **UX adaptada al rol**: misma app, interfaces distintas.

---

## Estructura

```text
src/app/
  core/           -- Auth, interceptors, guards, layout
  shared/         -- Componentes UI, pipes, modelos
  features/
    public-store/ -- Catalogo publico
      catalog/
      product-detail/
    customer/     -- Cliente autenticado (rol CUSTOMER)
      profile/
      orders/
      order-detail/
      checkout/   -- Redireccion a MP, resultado
    admin/        -- Backoffice (ADMIN/MANAGER/EMPLOYEE)
      dashboard/
      products/
      inventory/
      orders/
      pos/        -- Venta presencial (Point of Sale)
      cash/       -- Apertura, cierre, movimientos de caja
      suppliers/
      reports/
      users/
    auth/         -- Login y registro
```

---

## Rutas

```text
/store                    → Catalogo publico
/store/product/:id       → Detalle de producto

/auth/login              → Login
/auth/register           → Registro

/cart                    → Carrito local

/customer/orders         → Pedidos
/customer/orders/:id     → Detalle de pedido
/customer/profile        → Perfil

/admin                   → Dashboard
  /admin/products        → Productos
  /admin/stock           → Stock y lotes
  /admin/orders          → Pedidos
  /admin/pos             → Venta presencial (Point of Sale)
  /admin/cash            → Caja (apertura/cierre/movimientos)
  /admin/suppliers       → Proveedores
  /admin/reports         → Reportes y recomendaciones
  /admin/users           → Usuarios internos
```

---

## Carrito (localStorage)

El carrito se maneja en frontend con localStorage.

```typescript
@Injectable({ providedIn: 'root' })
class CartService {
  private localStorageKey = 'lembas_cart';
  private items = signal<CartItem[]>(this.loadFromStorage());

  getItems() { return this.items.asReadonly(); }

  addItem(product, quantity, branchId) { ... this.saveToStorage(); }
  removeItem(productId) { ... this.saveToStorage(); }
  clear() { this.items.set([]); localStorage.removeItem(this.localStorageKey); }

  private loadFromStorage(): CartItem[] {
    return JSON.parse(localStorage.getItem(this.localStorageKey) || '[]');
  }
  private saveToStorage() {
    localStorage.setItem(this.localStorageKey, JSON.stringify(this.items()));
  }
}
```

---

## Flujo de checkout con Mercado Pago

```text
1. Cliente confirma compra
2. Frontend llama POST /api/customer/orders
3. Backend crea orden (PENDING_PAYMENT) y payment (PENDING)
4. Frontend recibe orderId
5. Frontend llama POST /api/customer/orders/{orderId}/checkout/mp
6. Backend crea preferencia en MP
7. Frontend recibe initPoint
8. Frontend redirige a MP Checkout Pro (window.location.href = initPoint)
9. MP redirige al frontend (splash de resultado)
10. Frontend consulta GET /api/customer/orders/{orderId} para estado final
11. MP envia webhook al backend (procesa en background)
```

---

## Guards

| Guard | Rutas | Logica |
|---|---|---|
| `AuthGuard` | `/customer/**`, `/admin/**` | Redirige a `/auth/login` |
| `AdminGuard` | `/admin/**` | Requiere ADMIN/MANAGER/EMPLOYEE |

---

> [!seealso] Notas relacionadas
> - [[02 - Modulos/Tienda Online]]
> - [[02 - Modulos/Backoffice]]
> - Volver a [[_Index]]
