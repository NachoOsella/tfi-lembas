---
title: "Seguridad y Roles"
tags:
  - requisitos
  - seguridad
  - roles
  - permisos
  - rbac
---

# Seguridad y Roles

> [!info] Autenticacion y autorizacion basada en roles fijos para el MVP. Los permisos granulares quedan como evolucion futura.

---

## Autenticacion

- Login con email y contrasena
- Password hasheada con algoritmo seguro (BCrypt)
- Tokens JWT para sesiones
- Refresh token opcional

---

## Roles del sistema

Se definen cuatro roles:

| Rol | Descripcion |
|---|---|
| `ADMIN_GENERAL` | Acceso total al sistema y configuracion del negocio |
| `ENCARGADO_SUCURSAL` | Administra operacion de una sucursal especifica |
| `EMPLEADO` | Opera ventas, pedidos, stock basico y tareas asignadas |
| `CLIENTE` | Solo acceso a tienda online para comprar (para MVP se usa checkout invitado; el rol CLIENTE es opcional y aplica si se implementa registro de clientes en el futuro) |

---

## Matriz de acceso por rol

| Modulo | Admin General | Encargado Sucursal | Empleado | Cliente |
|---|---:|---:|---:|---:|
| Catalogo publico | Si | Si | Si | Si |
| Comprar online (checkout invitado) | -- | -- | -- | Si |
| Productos (ABM) | Si | Parcial | No | No |
| Stock (gestion) | Si | Si | Parcial (consulta) | No |
| Ventas presenciales | Si | Si | Si | No |
| Pedidos (todos) | Si | Si | Solo preparacion | Solo propios |
| Proveedores | Si | Parcial (consulta) | No | No |
| Precios | Si | Parcial (consulta) | No | No |
| Analytics | Si (global) | Si (sucursal) | No/parcial | No |
| Usuarios | Si | Parcial (sucursal) | No | No |
| Configuracion | Si | No/parcial | No | No |

---

## Permisos por accion

### Administrador general

- Gestionar empresa
- Gestionar sucursales
- Gestionar usuarios y asignacion de roles fijos
- Gestionar productos, categorias, marcas
- Gestionar proveedores
- Ver todas las ventas (global)
- Ver reportes globales
- Configurar margenes y reglas comerciales
- Aprobar cambios masivos de precio (post-MVP, cuando se implemente importacion automatica)
- Ver analytics completos

### Encargado de sucursal

- Ver stock de su sucursal
- Gestionar ventas de su sucursal
- Gestionar pedidos asignados a su sucursal
- Preparar entregas/retiros
- Registrar ajustes de stock justificados
- Ver reportes de su sucursal
- Imprimir etiquetas

### Empleado

- Realizar ventas presenciales
- Buscar productos
- Escanear codigos de barras
- Preparar pedidos
- Registrar consumo interno (si esta permitido)
- Reportar mermas o problemas
- Consultar stock basico

### Cliente

- Ver catalogo
- Buscar productos
- Agregar al carrito
- Realizar pedido
- Pagar mediante link
- Consultar estado del pedido
- Recibir recomendaciones

---

## Auditoria

Acciones que deben quedar registradas con fecha, usuario y motivo:

- Cambio de precio
- Cambio masivo de precios
- Ajuste manual de stock
- Cancelacion de pedido
- Cambio de estado de pago
- Creacion/desactivacion de usuarios
- Cambio de rol de usuario
- Registro de merma/vencimiento

---

## Reglas de seguridad adicionales

- El sistema no almacena datos sensibles de tarjetas de credito/debito.
- El asistente IA no debe exponer datos internos al cliente (margenes, costos).
- El asistente IA no debe hacer diagnosticos medicos ni prometer beneficios de salud.
- Las contrasenas deben cumplir con politicas de complejidad minima.

---

> [!seealso] Notas relacionadas
> - [[Funcionales]] -- requisitos funcionales asociados (RF-050 a RF-054)
> - [[02 - Modulos/Asistente Inteligente]] -- reglas de seguridad del asistente
> - Volver a [[_Index]]
