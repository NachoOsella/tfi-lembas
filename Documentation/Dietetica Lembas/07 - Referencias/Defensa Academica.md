---
title: "Argumento de Defensa Academica"
tags:
  - referencia
  - defensa
  - tesis
  - argumento
---

# Argumento de Defensa Academica

> [!info] Puntos clave para defender el proyecto ante un tribunal academico.

---

## Por que no es un CRUD

El sistema no se limita a ABMs porque implementa **procesos completos**:

- Compra online con validacion de stock en tiempo real
- Pedido con maquina de estados
- Pago con estado asociado y trazabilidad
- Preparacion y entrega con cambios de estado
- Venta presencial con descuento automatico de stock
- Gestion de stock por sucursal con lotes y vencimientos
- Trazabilidad completa de movimientos de stock
- Actualizacion asistida de precios con aprobacion humana
- Recomendaciones inteligentes basadas en datos reales
- Analytics comercial con metricas accionables

---

## Por que no es solo un e-commerce

El proyecto supera un e-commerce basico porque integra:

- **Backoffice real** para operacion diaria del negocio
- **Registro de clientes** con rol CUSTOMER y autenticacion JWT
- **Stock por lotes con FEFO** y trazabilidad completa de movimientos
- Gestion de proveedores con costos manuales
- Stock por sucursal con control de vencimientos
- Ventas presenciales con scanner y descuento de stock
- Reposicion sugerida basada en ventas y stock
- Reportes basicos con metricas del negocio
- Roles y permisos diferenciados (ADMIN, MANAGER, EMPLOYEE, CUSTOMER)
- Auditoria de acciones criticas

---

## Por que tiene valor para un cliente real

Porque resuelve problemas concretos de un comercio real:

- Vender online **sin perder control de stock**
- Reducir trabajo manual y errores operativos
- Mejorar visibilidad de pedidos y entregas
- Mantener precios actualizados desde listas de proveedores
- **Evitar vender productos agotados** (stock sincronizado)
- Detectar productos criticos (bajo stock, vencimiento)
- Agilizar venta presencial con codigo de barras
- Mejorar toma de decisiones con datos comerciales

---

## Por que es tecnicamente defendible

Porque involucra conceptos tecnicos relevantes:

- **Arquitectura modular** (monolito con separacion por features)
- **Seguridad y roles** (RBAC, JWT, autenticacion)
- **Transacciones** para consistencia de stock y pedidos
- **Modelado de dominio real** con entidades, reglas de negocio y estados
- **Integraciones mediante adaptadores** (pagos, IA, etiquetas)
- **Testing de flujos criticos** (unit, integration, E2E)
- **Diseno frontend por features** con lazy loading
- **Analytics** con metricas comerciales
- **Asistente inteligente** controlado por reglas del sistema (sin invencion, basado en datos reales del catalogo y stock)

---

## Texto corto para presentar el alcance

> [!quote] El proyecto consiste en una sistema integral de gestion comercial con modulo e-commerce para Dietetica Lembas, orientada a permitir la venta online de productos con stock real por sucursal, gestion de pedidos, pagos mediante QR o link de pago, retiro o envio, y seguimiento de estados. La solucion incluye un backoffice operativo para administrar productos, stock, proveedores, listas de precios, ventas presenciales, empleados, sucursales, impresion de etiquetas, codigos de barras y reportes comerciales. Ademas, incorpora un modulo inteligente capaz de recomendar productos al cliente y sugerir acciones comerciales al administrador en base a stock, ventas, vencimientos y disponibilidad.

---

## Texto formal para documento academico

> Se propone el desarrollo de una plataforma web de comercio electronico para una dietetica real, complementada con un modulo de administracion interna que permita operar de forma integrada el catalogo, el stock, los pedidos, los pagos, las entregas, los proveedores y las ventas presenciales. El sistema busca resolver la desconexion habitual entre el canal online y la gestion operativa del comercio, evitando inconsistencias de stock, facilitando la actualizacion de precios y brindando informacion comercial para la toma de decisiones. Como elemento diferencial, se incorpora un asistente inteligente que genera recomendaciones de productos y sugerencias operativas utilizando informacion real del catalogo, disponibilidad, vencimientos y comportamiento de ventas.

---

> [!seealso] Notas relacionadas
> - Volver a [[_Index]]
