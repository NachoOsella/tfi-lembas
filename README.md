# TFI Lembas

Technical documentation for **Lembas**, an internal web system for a single-branch health food store.

<p align="center">
  <img src="Documentation/attachments/lembas-login.png" alt="Lembas - login screen" width="720" />
</p>

> [!NOTE]
> This repository primarily contains **functional, technical, and planning documentation** for the system.

## Overview

Lembas centralizes key store operations in a single application:

- Product management
- Lot- and expiration-based stock control
- Counter sales flow
- Cash register tracking and traceability
- Suppliers and simple purchasing flow
- Price lists and assisted price updates
- Promotions and label generation
- Reporting, alerts, and audit trail

The main goal is to replace spreadsheets and scattered records with a system that enforces explicit business rules and improves operational consistency.

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

If you are new to the project, start with:

1. `Documentation/00 - Vision general.md`
2. `Documentation/01 - Contexto y problema.md`
3. `Documentation/02 - Alcance del sistema.md`

Then continue by area depending on your role:

- **Functional analysis**: `03 - Requisitos/`, `05 - Casos de uso/`
- **Domain understanding**: `04 - Dominio/`
- **Technical design**: `06 - Arquitectura/`, `07 - API/`
- **UX/UI perspective**: `08 - UX UI/`
- **QA and validation**: `09 - Testing y calidad/`
- **Planning and roadmap**: `10 - Planificacion/`

## Documentation modules

| Module | Purpose |
|---|---|
| `03 - Requisitos` | Functional/non-functional requirements and traceability matrix |
| `04 - Dominio` | Business rules, conceptual model, domain glossary, lifecycle definitions |
| `05 - Casos de uso` | Actors, primary use cases, and critical flows |
| `06 - Arquitectura` | Architectural decisions, backend/frontend design, database, security, deployment |
| `07 - API` | Endpoint catalog and DTO contracts |
| `08 - UX UI` | Roles, key screens, user flows, and visual references |
| `09 - Testing y calidad` | Testing strategy, test cases, and critical validations |
| `10 - Planificacion` | MVP definition, sprint plan, and future roadmap |
| `11 - Anexos` | Supporting material and presentation artifacts |

## Visual references

The repository includes UI captures in:

- `Documentation/attachments/lembas-login.png`
- `Documentation/attachments/lembas-home-admin.png`
- `Documentation/attachments/lembas-home-empleado-refinada.png`

## Documentation conventions

- Files are split by area to keep content maintainable and easy to navigate.
- Requirements traceability is centralized in:  
  `Documentation/03 - Requisitos/03.3 - Matriz de trazabilidad.md`
- Images are linked with relative paths for portability.

> [!TIP]
> For better navigation of wiki-style links (`[[...]]`) in the docs, use Obsidian.

## Scope snapshot

Lembas is intentionally scoped as an internal management system.

> [!IMPORTANT]
> It is **not** intended to be a full fiscal POS or an advanced accounting platform. This boundary keeps the TFI focused, feasible, and defendable.
