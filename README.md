# TFI Lembas

Documentación técnica del proyecto **Lembas**, un sistema web interno para una dietética de una sola sucursal.

<p align="center">
  <img src="Documentation/attachments/lembas-login.png" alt="Lembas - pantalla de login" width="720" />
</p>

> [!NOTE]
> Este repositorio contiene principalmente **documentación funcional, técnica y de planificación** del sistema.

## Overview

Lembas centraliza operaciones clave del negocio en una sola aplicación:

- Gestión de productos
- Stock por lote y vencimiento
- Ventas de mostrador
- Caja y trazabilidad
- Proveedores y compras simples
- Listas y actualización asistida de precios
- Ofertas y etiquetas
- Reportes, alertas y auditoría

El objetivo es reemplazar planillas y registros dispersos por un sistema con reglas de negocio explícitas y mayor consistencia operativa.

## Repository structure

```text
.
├── Documentation/
│   ├── 00 - Vision general.md
│   ├── 01 - Contexto y problema.md
│   ├── 02 - Alcance del sistema.md
│   ├── 03 - Requisitos/
│   ├── 04 - Dominio/
│   ├── 05 - Casos de uso/
│   ├── 06 - Arquitectura/
│   ├── 07 - API/
│   ├── 08 - UX UI/
│   ├── 09 - Testing y calidad/
│   ├── 10 - Planificacion/
│   ├── 11 - Anexos/
│   └── attachments/
└── README.md
```

## Recommended reading path

If you are new to the project, start here:

1. `Documentation/00 - Vision general.md`
2. `Documentation/01 - Contexto y problema.md`
3. `Documentation/02 - Alcance del sistema.md`

Then continue by module depending on your role:

- **Functional analysis**: `03 - Requisitos/`, `05 - Casos de uso/`
- **Domain understanding**: `04 - Dominio/`
- **Technical design**: `06 - Arquitectura/`, `07 - API/`
- **UX perspective**: `08 - UX UI/`
- **QA and validation**: `09 - Testing y calidad/`
- **Planning and roadmap**: `10 - Planificacion/`

## Documentation modules

| Module | Purpose |
|---|---|
| `03 - Requisitos` | Functional/non-functional requirements and traceability matrix |
| `04 - Dominio` | Business rules, conceptual model, domain glossary, lifecycle definitions |
| `05 - Casos de uso` | Actors, core use cases and critical flows |
| `06 - Arquitectura` | Architectural decisions, backend/frontend design, DB, security, deployment |
| `07 - API` | Endpoint catalog and DTO contracts |
| `08 - UX UI` | Roles, key screens, user flows and visual references |
| `09 - Testing y calidad` | Testing strategy, test cases and critical validations |
| `10 - Planificacion` | MVP boundaries, sprint plan and future roadmap |
| `11 - Anexos` | Supporting material and presentation artifacts |

## Visual references

The project includes UI captures in:

- `Documentation/attachments/lembas-login.png`
- `Documentation/attachments/lembas-home-admin.png`
- `Documentation/attachments/lembas-home-empleado-refinada.png`

## Working with the docs

- Markdown files are modular and focused to keep navigation fast.
- Requirement traceability is centralized in:  
  `Documentation/03 - Requisitos/03.3 - Matriz de trazabilidad.md`
- Images are linked using relative paths for portability.

> [!TIP]
> For local navigation with wikilinks (`[[...]]`), use Obsidian over a plain Markdown editor.

## Scope snapshot

Lembas is intentionally scoped as an internal management system.

> [!IMPORTANT]
> It is **not** intended to be a full fiscal POS or advanced accounting platform. This boundary keeps the TFI focused, feasible, and defendable.
