---
title: "Seguridad y Roles"
tags:
  - requisitos
  - seguridad
  - roles
  - permisos
---

# Seguridad y Roles

> [!info] Autenticacion JWT y autorizacion basada en roles fijos.

---

## Autenticacion

- Registro de clientes (rol CUSTOMER)
- Login con email y contrasena
- Password hasheada con BCrypt
- Tokens JWT (24h de duracion)

---

## Roles del sistema

| Rol | Descripcion | branch_id |
|---|---|---|
| `ADMIN` | Acceso total al sistema | Opcional |
| `MANAGER` | Gestion operativa de sucursal | Obligatorio |
| `EMPLOYEE` | Ventas, preparacion, consulta stock, caja | Obligatorio |
| `CUSTOMER` | Cliente registrado que compra online | Debe ser null |

---

## Matriz de acceso por rol

| Modulo | ADMIN | MANAGER | EMPLOYEE | CUSTOMER |
|---|---:|---:|---:|---:|
| Catalogo publico | Si | Si | Si | Si |
| Comprar online | -- | -- | -- | Si |
| Productos (ABM) | Si | Parcial | No | No |
| Stock (gestion) | Si | Si | Consulta | No |
| Caja (abrir/cerrar) | Si | Si | Si | No |
| Ventas presenciales (POS) | Si | Si | Si | No |
| Ordenes | Si | Si | Preparacion | Solo propias |
| Proveedores | Si | Consulta | No | No |
| Reportes | Si (global) | Si (sucursal) | No | No |
| Usuarios internos | Si | Parcial | No | Solo su perfil |
| Pagos online (MP) | -- | -- | -- | Si |
| Webhooks MP | -- | -- | -- | -- |

---

## Permisos detallados

### ADMIN
- Gestionar usuarios, sucursales, productos, proveedores
- Abrir y cerrar caja
- Ver y gestionar todas las cajas
- Ver reportes completos
- Auditar eventos

### MANAGER
- Abrir y cerrar caja de su sucursal
- Gestionar ventas presenciales
- Registrar movimientos de caja
- Ver reportes operativos de su sucursal
- Preparar ordenes online

### EMPLOYEE
- Abrir y cerrar caja (queda auditado)
- Registrar venta presencial (requiere caja abierta)
- Registrar pagos presenciales
- Consultar stock
- Preparar pedidos online
- Registrar movimientos simples de caja
- Si cierra caja con diferencia, debe explicar obligatoriamente

### CUSTOMER
- Registrarse e iniciar sesion
- Ver catalogo
- Usar carrito local
- Crear orden online
- Pagar con Mercado Pago Checkout Pro
- Consultar sus pedidos
- NO puede abrir/cerrar/consultar caja

---

## Auditoria

Eventos registrados en `audit_logs` con usuario, fecha y descripcion:

- Cambio de precio de venta
- Ajuste manual de stock
- Cancelacion de orden
- Apertura y cierre de caja
- Confirmacion de pago (webhook MP)
- Movimientos de caja
- Alta/baja de usuarios internos

---

> [!seealso] Notas relacionadas
> - [[Funcionales]]
> - Volver a [[_Index]]
