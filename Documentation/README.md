---
title: Documentacion Lembas
tags:
  - tfi
  - lembas
  - indice
aliases:
  - Indice Lembas
---

# Documentacion tecnica del TFI Lembas

> [!abstract]
> Carpeta de documentacion modular del sistema **Lembas**, una aplicacion web interna para una dietetica de una sola sucursal.

## Inicio recomendado

- [[00 - Vision general]]
- [[01 - Contexto y problema]]
- [[02 - Alcance del sistema]]

## Indice por area

| Area | Documentos |
|---|---|
| Requisitos | [[03.1 - Requisitos funcionales]], [[03.2 - Requisitos no funcionales]], [[03.3 - Matriz de trazabilidad]] |
| Dominio | [[04.1 - Reglas de negocio]], [[04.2 - Modelo conceptual]], [[04.3 - Estados y ciclos de vida]], [[04.4 - Glosario del dominio]] |
| Casos de uso | [[05.1 - Actores]], [[05.2 - Casos de uso principales]], [[05.3 - Flujos criticos]] |
| Arquitectura | [[06.1 - Decision arquitectonica]], [[06.2 - Backend]], [[06.3 - Frontend]], [[06.4 - Base de datos]], [[06.5 - Seguridad]], [[06.6 - Despliegue]] |
| API | [[07.1 - Endpoints]], [[07.2 - Contratos DTO]] |
| UX/UI | [[08.1 - Roles y pantallas]], [[08.2 - Flujos de usuario]], [[08.3 - Referencias visuales]] |
| Calidad | [[09.1 - Estrategia de testing]], [[09.2 - Casos de prueba]], [[09.3 - Validaciones criticas]] |
| Planificacion | [[10.1 - MVP]], [[10.2 - Plan de sprints]], [[10.3 - Roadmap futuro]] |
| Anexos | [[Presentacion para la duena]], [[Imagenes]] |

## Criterios de mantenimiento

- El documento central no duplica el detalle de cada modulo: enlaza a los documentos especializados.
- Los requisitos se trazan contra reglas de negocio, casos de uso y pruebas en [[03.3 - Matriz de trazabilidad]].
- Los nombres de archivos evitan acentos para mejorar portabilidad entre sistemas operativos.
- Las imagenes se referencian con rutas relativas desde esta carpeta.
- Las carpetas locales de herramientas, como `.obsidian` y `.pi`, no forman parte del entregable versionado.
