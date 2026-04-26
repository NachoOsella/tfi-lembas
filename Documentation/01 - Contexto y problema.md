---
title: Contexto y problema
tags:
  - tfi
  - analisis
  - contexto
aliases:
  - Problema de negocio
---

# Contexto y problema

## Contexto del negocio

Lembas se plantea para una dietetica de una sola sucursal que opera con productos empaquetados, productos sin codigo de barras, productos con vencimiento, proveedores diversos, precios variables y tareas cotidianas de mostrador. La operatoria puede apoyarse en planillas, mensajes de WhatsApp, listas de proveedores, controles visuales y anotaciones manuales.

Este esquema resulta suficiente en etapas iniciales, pero dificulta mantener consistencia cuando se necesita saber stock real, precios vigentes, vencimientos proximos, caja del turno, consumos internos y cambios aplicados por cada usuario.

## Situacion problematica

El problema principal es la falta de una plataforma integrada que conecte catalogo, stock, venta, caja, proveedores y auditoria. Esto genera riesgos operativos:

- dependencia de Excel y registros dispersos;
- stock fisico y stock registrado desalineados;
- dificultad para aplicar FEFO en productos con vencimiento;
- precios de venta o etiquetas desactualizadas;
- listas de proveedores con formatos heterogeneos;
- cambios de costo sin comparacion historica clara;
- ventas mostrador que no descuentan stock automaticamente;
- caja poco vinculada con las ventas confirmadas;
- consumo interno, merma y vencimiento registrados de forma confusa;
- empleados con posible acceso a informacion sensible que no necesitan;
- falta de auditoria sobre acciones criticas.

## Preguntas que el sistema debe responder

| Pregunta | Area relacionada |
|---|---|
| Que stock real hay de cada producto? | Stock |
| Que lote vence primero? | Stock FEFO |
| Que productos deben reponerse? | Stock y proveedores |
| Que proveedor tiene el costo mas conveniente actualizado? | Proveedores y listas |
| Que precios cambiaron y que etiquetas faltan imprimir? | Pricing y etiquetas |
| Que ventas hizo un empleado en su turno? | Ventas y caja |
| Que ingreso de caja corresponde a cada venta? | Caja |
| Que productos se consumieron internamente? | Consumo interno |
| Quien modifico un precio, stock o cierre? | Auditoria |

## Objetivo general

Diseñar e implementar una aplicacion web interna que centralice la gestion operativa de la dietetica, integrando productos, stock por lote y vencimiento, proveedores, compras simples, listas de precios, actualizacion asistida, venta mostrador simplificada, caja, consumo interno, ofertas, etiquetas, reportes y auditoria.

## Objetivos especificos

1. Centralizar datos en PostgreSQL.
2. Diferenciar roles de administradora/dueña y empleado.
3. Gestionar catalogo, categorias, marcas, presentaciones y codigos de barras.
4. Registrar stock por lotes y vencimientos.
5. Descontar stock con criterio FEFO.
6. Registrar ventas mostrador con scanner o busqueda manual.
7. Integrar venta confirmada con egreso de stock e ingreso de caja.
8. Diferenciar ventas, consumos internos, mermas, vencimientos y ajustes.
9. Gestionar proveedores y mapeos de producto interno contra producto de proveedor.
10. Cargar listas por Excel, texto de WhatsApp o carga manual.
11. Comparar costos, sugerir precios y exigir aprobacion para aplicar cambios.
12. Registrar compras simples a proveedores para saber que productos ingresaron y generar stock.
13. Marcar etiquetas pendientes cuando cambian precios u ofertas.
13. Proveer reportes operativos y alertas.
14. Auditar acciones criticas.

## Valor esperado

| Beneficio | Impacto |
|---|---|
| Centralizacion | Menor dependencia de planillas y mensajes sueltos. |
| Trazabilidad | Posibilidad de reconstruir acciones y responsables. |
| Stock confiable | Mejor control de vencimientos, mermas y reposicion. |
| Venta agil | Uso de scanner sin convertir el proyecto en POS fiscal. |
| Caja ordenada | Relacion directa entre venta confirmada e ingreso. |
| Precios controlados | Revision humana antes de aplicar cambios masivos. |
| Roles claros | Cada usuario ve y opera solo lo necesario. |

## Criterio academico

El TFI es defendible si demuestra un dominio realista, reglas claras, consistencia transaccional, arquitectura modular, seguridad por roles, pruebas sobre procesos criticos y documentacion trazable entre requisitos, reglas, casos de uso y validaciones.
