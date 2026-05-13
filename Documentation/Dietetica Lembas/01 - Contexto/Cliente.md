---
title: "Contexto del Cliente"
tags:
  - contexto
  - cliente
  - dietetica-lembas
---

# Contexto del Cliente

> [!info] **Dietetica Lembas** es el comercio real tomado como caso de estudio para validar el sistema.

## El negocio

Comercializa productos tipicos de dietetica:

- Alimentos saludables y naturales
- Suplementos alimenticios no farmacologicos
- Productos sin TACC, sin azucar
- Productos a granel
- Snacks, cereales, frutos secos, harinas
- Infusiones y productos relacionados

> [!note] Importante
> El sistema se disena tomando a Dietetica Lembas como cliente real, pero la arquitectura y el dominio deben permitir adaptarlo a otros comercios minoristas.
>
> **Decision de diseno:** aunque el caso inicial puede operar con una unica sucursal, el dominio se modela con soporte multisucursal desde el inicio. Esto evita redisenos futuros y permite demostrar separacion de stock, ventas, empleados y pedidos por ubicacion como valor academico del diseno.

## Situacion actual esperada

El negocio enfrenta problemas frecuentes en comercios minoristas:

| Problema                                                    | Impacto                                |
| ----------------------------------------------------------- | -------------------------------------- |
| Administracion manual o semi-manual de stock                | Errores operativos, perdida de tiempo  |
| Uso de planillas Excel para productos, precios, proveedores | Datos descentralizados, inconsistentes |
| Dificultad para mantener precios actualizados               | Perdida de margen o clientes           |
| Proveedores con formatos heterogeneos de listas de precios  | Carga manual lenta, errores            |
| Falta de trazabilidad de movimientos de stock               | No se puede auditar ni corregir        |
| Dificultad para saber que productos reponer                 | Roturas de stock no detectadas         |
| Posibles diferencias entre stock real y registrado          | Inconsistencias que se acumulan        |
| Falta de canal online integrado con stock real              | Perdida de ventas online               |
| Permisos no diferenciados entre dueña, admins y empleados   | Riesgo de seguridad                    |
| Falta de metricas comerciales claras                        | Decisiones sin datos                   |

## Oportunidad

Transformar un sistema interno de gestion en una **plataforma comercial completa**:

- Venta online con catalogo publico
- Stock sincronizado por sucursal
- Pedidos con retiro o envio
- Pagos digitales
- Backoffice para operacion diaria
- Analytics para decisiones comerciales
- IA para recomendaciones y sugerencias

---

> [!seealso] Notas relacionadas
> - [[Problematica]] -- problemas detallados que resuelve el sistema
> - [[Propuesta]] -- solucion propuesta
> - Volver a [[_Index]]
