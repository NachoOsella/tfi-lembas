---
title: "Diseno Frontend"
tags:
  - arquitectura
  - frontend
  - angular
  - componentes
  - arquitectura-detallada
---

# Diseno Frontend

> [!info] Arquitectura detallada del frontend Angular: estructura de componentes, estado global, routing, diseno de pantallas y estilos.

---

## Principios arquitectonicos

1. **Separacion de concerns**: los componentes de UI no contienen logica de negocio. La logica vive en servicios especificos de cada feature.
2. **Estado descentralizado pero consistente**: no hay un store global gigante. Cada feature gestiona su estado interno con Signals. Solo el estado compartido (usuario autenticado, sucursal seleccionada, carrito) es global.
3. **Lazy loading por feature**: el usuario no descarga codigo que no usa. El backoffice (pesado) se carga solo cuando el empleado se loguea.
4. **Componentes compartidos genericos**: los componentes de `shared/ui` no conocen el dominio. Son puramente visuales y reciben datos por @Input.
5. **Manejo de errores centralizado**: un interceptor HTTP global captura errores y decide la accion (toast, redirect, alerta) segun el codigo de error.
6. **UX adaptada al rol**: la misma aplicacion Angular renderiza interfaces radicalmente distintas segun el rol del usuario (cliente ve tienda, empleado ve backoffice).

---

## Arquitectura de aplicaciones (dos caras de la misma moneda)

El frontend es una unica aplicacion Angular con dos "modos" visuales:

```text
src/app/
  core/           -- Compartido global (auth, layout, interceptors, guards)
  shared/         -- Componentes UI puros y pipes/directivas
  features/
    shop/         -- Tienda online (publica, para CLIENTE)
      catalog/
      product-detail/
      cart/
      checkout/
      order-tracking/
      assistant-widget/
    admin/        -- Backoffice (protegida, para ADMIN/ENCARGADO/EMPLEADO)
      dashboard/
      products/
      inventory/
      suppliers/
      sales/      -- Venta rapida
      orders/
      deliveries/
      labels/
      analytics/
      users/
      settings/
    auth/         -- Login (ambos modos comparten)
```

**Por que una unica aplicacion y no dos (shop.site.com + admin.site.com)**: compartir el modulo de autenticacion, el interceptor HTTP, los componentes compartidos y el layout global es mucho mas simple en un solo proyecto. La separacion visual se logra con routing y guards. Ademas, compartir codigo entre tienda y backoffice (como el selector de sucursales o el buscador de productos) es natural en un solo proyecto.

**Trade-off**: el bundle inicial es mas grande que si fueran aplicaciones separadas. Pero con lazy loading bien configurado, el usuario solo descarga lo que necesita en cada momento.

---

## Arquitectura de routing

### Estructura de rutas (Angular Router)

```text
/                           → Redirect a /shop
/shop                       → Layout publico (header + footer)
  /shop/catalog             → Catalogo de productos
  /shop/catalog/:id         → Detalle de producto
  /shop/cart                → Carrito de compras
  /shop/checkout            → Checkout (requiere carrito no vacio)
  /shop/order/:orderNumber  → Seguimiento de pedido
  /shop/assistant           → Widget de asistente

/auth/login                 → Login (mismo componente para ambos roles)

/admin                      → Layout backoffice (sidebar + header)
  /admin/dashboard          → Panel principal
  /admin/products           → Lista de productos
  /admin/products/new       → Crear producto
  /admin/products/:id       → Editar producto
  /admin/inventory          → Stock y movimientos
  /admin/inventory/lots     → Lotes y vencimientos
  /admin/suppliers          → Proveedores
  /admin/sales              → Venta rapida (POS)
  /admin/sales/history      → Historial de ventas
  /admin/orders             → Pedidos online
  /admin/orders/:id         → Detalle de pedido
  /admin/deliveries         → Entregas
  /admin/labels             → Impresion de etiquetas
  /admin/analytics          → Reportes y dashboard
  /admin/users              → Usuarios internos
  /admin/settings           → Configuracion

/**                         → 404
```

### Estrategia de lazy loading

```typescript
const routes: Routes = [
  { path: '', redirectTo: '/shop', pathMatch: 'full' },
  {
    path: 'shop',
    loadChildren: () => import('./features/shop/shop.module').then(m => m.ShopModule)
    // Se carga cuando el usuario visita la tienda
  },
  {
    path: 'auth',
    loadChildren: () => import('./features/auth/auth.module').then(m => m.AuthModule)
    // Se carga cuando alguien hace login
  },
  {
    path: 'admin',
    loadChildren: () => import('./features/admin/admin.module').then(m => m.AdminModule),
    canActivate: [AuthGuard, AdminGuard]
    // Se carga solo cuando un empleado/admin autenticado accede
  }
];
```

**Por que lazy loading**: el modulo de administracion es grande (15+ pantallas). Sin lazy loading, un cliente que solo navega el catalogo descargaria todo el backoffice innecesariamente. Con lazy loading, el cliente descarga ~30% del codigo total.

### Guards

| Guard | Ruta | Logica |
|---|---|---|
| `AuthGuard` | `/admin/**` | Redirige a `/auth/login` si no hay JWT valido |
| `RoleGuard` | `/admin/users`, `/admin/settings` | Roles permitidos: `ADMIN_GENERAL` |
| `BranchGuard` | `/admin/sales`, `/admin/orders` | Verifica que el empleado tenga sucursal asignada |
| `CartGuard` | `/shop/checkout` | Redirige a `/shop/cart` si el carrito esta vacio |

---

## Arquitectura de estado global

### Que estado es global vs local

```text
ESTADO GLOBAL (core/services/)
  AuthService          ← Usuario autenticado, token, roles
  BranchService        ← Sucursal seleccionada (cliente) o asignada (empleado)
  CartService          ← Items del carrito (cliente)
  NotificationService  ← Toast/snackbar global

ESTADO LOCAL (cada feature/)
  CatalogService       ← Resultados de busqueda, filtros, paginacion
  ProductService       ← Producto en edicion, imagenes
  InventoryService     ← Stock actual, movimientos, alertas
  OrderService         ← Lista de pedidos, detalle, estados
  SaleService          ← Venta en curso (ticket temporal)
  SupplierService      ← Lista de proveedores
  AnalyticsService     ← Datos de dashboard y reportes
```

**Por que Signals y no NgRx o Akita**: para el 95% del estado de esta aplicacion, Signals de Angular es suficiente. El estado es mayormente local a una feature o a una pantalla. No hay flujos complejos de eventos entre features que justifiquen un store global como NgRx. Signals proporciona reactividad simple con mejor rendimiento que RxJS Subjects y menos boilerplate que NgRx.

**Excepcion**: el carrito (`CartService`) usa RxJS BehaviorSubject en lugar de Signals porque necesita reactividad entre componentes no relacionados (header muestra count, catalog agrega items, checkout consume). Pero incluso esto podria hacerse con Signals compartiendo el service.

### Patron de servicio con Signals

```typescript
// Cada servicio de feature expone:
@Injectable({ providedIn: 'root' })
class CatalogService {
  // Estado interno (privado)
  private _products = signal<Product[]>([]);
  private _loading = signal<boolean>(false);
  private _error = signal<string | null>(null);

  // Estado publico (solo lectura)
  readonly products = this._products.asReadonly();
  readonly loading = this._loading.asReadonly();
  readonly error = this._error.asReadonly();

  // Acciones
  search(query: string, branchId: number): void {
    this._loading.set(true);
    this._error.set(null);
    this.http.get(`/api/public/catalog/products?q=${query}&branchId=${branchId}`)
      .subscribe({
        next: (result) => { this._products.set(result); this._loading.set(false); },
        error: (err) => { this._error.set(err.message); this._loading.set(false); }
      });
  }
}
```

**Por que `providedIn: 'root'` para servicios de feature**: aunque es estado "local" a una feature, los servicios se registran en el root injector porque los componentes de una feature pueden estar en diferentes niveles de la jerarquia. Si se usara `providedIn` a nivel de modulo, habria que asegurarse de que el modulo se cargue antes que los componentes que lo usan, lo cual es fragil.

---

## Arquitectura de componentes

### Jerarquia de componentes

Cada pantalla sigue esta estructura:

```text
ShopLayoutComponent (header con sucursal + carrito count, footer)
  └── <router-outlet>
       ├── CatalogComponent (filtros + grilla)
       │    ├── ProductCardComponent (shared: imagen, nombre, precio, stock)
       │    ├── SearchInputComponent (shared: busqueda con debounce)
       │    ├── CategoryFilterComponent (shared: selector de categoria)
       │    └── PaginationComponent (shared)
       │
       ├── ProductDetailComponent
       │    ├── ImageGalleryComponent (galeria de imagenes)
       │    ├── QuantityInputComponent (shared: selector de cantidad)
       │    ├── BranchSelectorComponent (shared: selector de sucursal)
       │    └── ButtonComponent (shared: "Agregar al carrito")
       │
       ├── CartComponent
       │    ├── CartItemComponent (producto + cantidad + precio)
       │    └── CartSummaryComponent (subtotal, total, boton checkout)
       │
       └── CheckoutComponent
            ├── BranchSelectorComponent
            ├── DeliveryFormComponent (direccion para envio)
            └── OrderSummaryComponent (resumen + total + boton pagar)

AdminLayoutComponent (sidebar + header + breadcrumbs)
  └── <router-outlet>
       ├── DashboardComponent
       │    ├── AlertCardComponent (shared: alertas de stock bajo/vencimiento)
       │    ├── StatCardComponent (shared: metricas rapidas)
       │    └── QuickActionsComponent (accesos directos)
       │
       ├── ProductListComponent
       │    ├── DataTableComponent (shared: tabla con filtros)
       │    ├── SearchInputComponent
       │    └── StatusBadgeComponent (shared: publicado/pausado/borrador)
       │
       ├── ProductFormComponent
       │    ├── ImageUploaderComponent (subida con preview)
       │    ├── TagSelectorComponent (selector de etiquetas)
       │    └── SupplierCostSectionComponent (costo por proveedor)
       │
       ├── QuickSaleComponent (venta rapida POS)
       │    ├── BarcodeInputComponent (input con auto-focus para scanner)
       │    ├── TicketLineComponent (linea de producto + cantidad + precio)
       │    └── PaymentMethodSelectorComponent (selector de metodo de pago)
       │
       └── OrderDetailComponent
            ├── OrderTimelineComponent (linea de tiempo de estados)
            ├── OrderItemsComponent (lista de productos del pedido)
            └── StatusActionButtonsComponent (botones segun estado actual)
```

### Componentes compartidos (shared/ui)

| Componente | Inputs | Outputs | Uso principal |
|---|---|---|---|
| `ButtonComponent` | label, variant (primary/secondary/danger/ghost), size, disabled, loading | onClick | Toda la app |
| `DataTableComponent` | columns, data, pageSize, loading, sortable | sort, page | Listados del backoffice |
| `SearchInputComponent` | placeholder, debounceMs | search | Busqueda en catalogos y listados |
| `StatusBadgeComponent` | status, label | - | Estados de pedido, producto, pago |
| `ModalComponent` | title, size, closable | close, confirm | Confirmaciones, formularios |
| `ConfirmDialogComponent` | title, message, confirmLabel, cancelLabel | confirm, cancel | Acciones destructivas |
| `QuantityInputComponent` | value, min, max, step | valueChange | Carrito, venta, stock |
| `BranchSelectorComponent` | branches, selectedId | select | Tanto en shop como admin |
| `SkeletonComponent` | type (card/table/text), count | - | Loading states |
| `EmptyStateComponent` | icon, title, message, actionLabel | action | Tablas sin datos |
| `ToastComponent` | message, type (success/error/warning/info) | - | Notificaciones globales |
| `ImageUploaderComponent` | accept, maxSizeMb, currentImage | filesChanged | Formulario de producto |
| `MoneyInputComponent` | value, currency | valueChange | Precios en toda la app |

---

## Arquitectura de diseno visual

### Sistema de diseno (Design Tokens)

En lugar de un framework UI completo (PrimeNG, Angular Material), se propone un sistema de diseno propio basado en Tailwind CSS con tokens semanticos.

**Por que Tailwind y no un componente UI completo**: los frameworks de componentes UI (PrimeNG, Material) imponen su propio diseno y son pesados. Tailwind permite un diseno a medida con minimo CSS, es ligero en produccion (purga CSS no usado) y da control total sobre la apariencia. Para un proyecto academico donde el diseno debe ser propio, no usar un framework UI preconstruido es la decision correcta.

### Paleta de colores (semantica)

```text
Primary:    Verde natural (asociado a dietetica, saludable)
  - primary-50, primary-100, ..., primary-900

Secondary:  Marron/terracota (asociado a granos, natural)
  - secondary-50, ..., secondary-900

Semanticos:
  - success: Verde (stock ok, pago aprobado)
  - warning: Amarillo (stock bajo, vencimiento proximo)
  - danger:  Rojo (sin stock, pago rechazado)
  - info:    Azul (informacion, sugerencias)

Neutral:
  - white, gray-50 a gray-900, black
  - surface (fondos de tarjetas y paneles)
  - text-primary, text-secondary, text-muted
```

### Tipografia

```text
Headings:   Inter o system sans-serif (600/700 weight)
Body:       Inter o system sans-serif (400 weight)
Monospace:  JetBrains Mono o similar (codigos de barras, numeros de pedido)
Sizes:      12/14/16/18/20/24/30/36/48px
```

### Espaciado y layout

```text
Grid:       12 columnas responsive
Breakpoints: 640px (sm), 768px (md), 1024px (lg), 1280px (xl)
Container:  max-width 1280px centrado
Spacing:    4/8/12/16/20/24/32/40/48/64px
```

---

## Arquitectura de formularios

### Estrategia de formularios

Todos los formularios usan **Reactive Forms** de Angular (no template-driven). Motivo: validacion asincronica (verificar si un codigo de barras ya existe), formularios dinamicos (campos que aparecen/desaparecen segun el tipo de producto), y mejor testabilidad.

### Validaciones por formulario

| Formulario | Validaciones clave |
|---|---|
| Producto | nombre requerido, codigo barras unico (async), precio > 0, imagenes < 2MB |
| Venta | al menos un item, stock suficiente por item |
| Pedido | al menos un item, sucursal requerida, direccion si es envio |
| Usuario | email valido, password minima 8 caracteres |
| Proveedor | nombre requerido, email valido |
| Stock | cantidad > 0, motivo requerido en ajustes |
| Promocion | descuento entre 1% y 100%, fecha fin > fecha inicio |

---

## Arquitectura de manejo de errores en frontend

### Interceptor HTTP global

```text
HTTP Request
  ↓
AuthInterceptor
  ├── Agrega Authorization header si hay token
  └── Si el token expiro, intenta refresh (si falla, redirige a login)
  ↓
ErrorInterceptor
  ├── 401 → AuthService.logout() + redirect a /auth/login
  ├── 403 → NotificationService.warning("No tienes permisos")
  ├── 422/409 → NotificationService.error(mensaje del backend)
  ├── 500 → NotificationService.error("Error del servidor. Reintenta.")
  └── Network Error → NotificationService.error("Sin conexion al servidor")
  ↓
HTTP Response (o error manejado)
```

### Estados de vista unificados

Cada componente que carga datos desde el backend sigue este patron:

```typescript
// Tipo generico para todos los estados de carga
type ViewState<T> =
  | { status: 'idle' }                    // Nunca se ha cargado
  | { status: 'loading' }                  // Cargando...
  | { status: 'error'; message: string; code?: string; retry: () => void }
  | { status: 'empty'; message: string; suggestion?: string }
  | { status: 'data'; value: T };

// En el template:
@if (viewState().status === 'loading') {
  <app-skeleton type="table" [count]="5" />
} @else if (viewState().status === 'error') {
  <app-error-state [message]="viewState().message" (retry)="viewState().retry()" />
} @else if (viewState().status === 'empty') {
  <app-empty-state [message]="viewState().message" />
} @else if (viewState().status === 'data') {
  <!-- contenido real -->
}
```

**Por que 5 estados y no 3**: separar `idle` (nunca cargo) de `loading` (cargando ahora) permite mostrar skeletons solo cuando hay datos previos o mostrar un placeholder inicial. Separar `error` de `empty` permite mostrar mensajes distintos y boton de reintento solo en errores.

---

## Arquitectura de pantallas especificas

### Venta rapida (QuickSale) -- la pantalla mas critica

Esta es la pantalla mas usada del sistema (un empleado la usa cientos de veces al dia). Su diseno debe priorizar velocidad sobre todo.

**Principios de diseno**:
1. **Un solo campo activo**: el input de codigo de barras tiene auto-focus permanente. El empleado escanea y el producto se agrega automaticamente.
2. **Sin clicks innecesarios**: escanear un producto lo agrega al ticket inmediatamente. No hay "agregar al carrito" ni confirmacion.
3. **Cantidad editable in-situ**: al escanear el mismo producto dos veces, se incrementa la cantidad en el ticket. Se puede modificar con un input inline.
4. **Total visible siempre**: en la parte inferior, actualizado en tiempo real.
5. **Pago rapido**: al finalizar, se selecciona el metodo de pago y se confirma. 2-3 clicks maximo desde que se escanea el ultimo producto.

**Flujo de interaccion**:

```text
Pantalla abierta → input barcode con focus
  ├── Escanea "779123456" → producto aparece en ticket con cantidad=1
  ├── Escanea "779123456" otra vez → cantidad pasa a 2
  ├── Escanea "779654321" → segundo producto agregado
  ├── Click en cantidad de un item → input editable, cambia a 1
  │
  └── Click "Cobrar $5.500" → modal de metodos de pago
       ├── Selecciona "EFECTIVO" o "TRANSFERENCIA" o "QR"
       ├── (opcional) Ingresa monto recibido para calcular vuelto
       └── Confirma → venta registrada, ticket se limpia, focus vuelve al scanner
```

### Dashboard del administrador

**Principios de diseno**:
1. **Alertas visibles primero**: las tarjetas de alerta (stock bajo, vencimientos, pedidos pendientes) aparecen en la parte superior. Si no hay alertas, se muestra un estado tranquilo.
2. **Metricas clave en cards**: ventas hoy, pedidos pendientes, productos con bajo stock, productos proximos a vencer. Cada card con indicador de tendencia.
3. **Acciones rapidas**: botones para las tareas mas frecuentes (nueva venta, nuevo producto, imprimir etiquetas, ver pedidos).
4. **Acceso a reportes**: enlaces a analytics detallados.

---

## Arquitectura de performance

### Carga inicial

```text
Primera carga:
  - Solo modulo shop (catalogo publico) + auth
  - Aproximadamente 200-300KB JS comprimido
  - Lazy loading del resto

Despues del login como admin:
  - Carga modulo admin (diferido, ~400-500KB)
  - Carga solo si el rol tiene acceso

Cache:
  - Catalogo publico cacheable (CDN o Service Worker)
  - Imagenes de producto con Cache-Control: public, max-age=86400
```

### Optimizaciones clave

1. **ChangeDetectionStrategy.OnPush** en todos los componentes. Solo se actualizan cuando cambia una @Input o un signal.
2. **`trackBy`** en todos los *ngFor para evitar recrear el DOM al modificar listas.
3. **Virtual scrolling** para tablas largas (DataTableComponent con CDK virtual scroll si supera 100 filas).
4. **Debounce en busquedas**: SearchInputComponent con 300ms de debounce.
5. **Imagenes lazy loading**: `loading="lazy"` nativo en todas las imagenes del catalogo.
6. **Precarga de modulos**: despues de la carga inicial, precargar el modulo admin en background si el usuario tiene rol de empleado.

---

## Decisiones de UX especificas

| Decision | Por que |
|---|---|
| Backoffice y tienda en misma URL base pero rutas distintas | Comparten auth, componentes, servicios. Mas simple que dos aplicaciones separadas. |
| Venta rapida con foco permanente en scanner | El empleado no debe tocar el mouse entre productos escaneados. |
| Confirmacion explicita solo para acciones destructivas | Cancelar pedido, eliminar producto, ajustar stock. Agregar al carrito no necesita confirmacion. |
| Carrito persistente en sesion (no localStorage) | Si el usuario cierra el navegador, el carrito se pierde (evita acumular productos de semanas anteriores). |
| Toast notificaciones con auto-dismiss | Errores no bloqueantes desaparecen en 5 segundos. Acciones confirmadas muestran feedback momentaneo. |
| Sidebar colapsable en backoffice | Para empleados en pantallas chicas o que necesitan maximo espacio vertical en venta rapida. |

---

> [!seealso] Notas relacionadas
> - [[Vision General]] -- stack tecnologico
> - [[02 - Modulos/Tienda Online]] -- funcionalidades del modulo e-commerce
> - [[02 - Modulos/Backoffice]] -- funcionalidades del modulo administracion
> - [[02 - Modulos/Asistente Inteligente]] -- widget de recomendaciones
> - Volver a [[_Index]]
