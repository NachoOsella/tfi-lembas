---
title: "Vision General de Arquitectura"
tags:
  - arquitectura
  - vision-general
  - stack
---

# Vision General de Arquitectura (MVP Simplificado)

> [!info] Monolito modular con separacion por features.

---

## Estilo arquitectonico

```text
Frontend Angular (tienda + backoffice)
    ↓ HTTP REST
Backend Spring Boot (modulos por feature)
    ↓ JPA/Hibernate
PostgreSQL
```

## Diagrama de arquitectura

```mermaid
%%{init: {"theme": "base", "themeVariables": {"darkMode": true, "background": "#1d2021", "primaryColor": "#32302f", "primaryTextColor": "#d4be98", "primaryBorderColor": "#d4be98", "lineColor": "#d4be98", "secondaryColor": "#32302f", "tertiaryColor": "#1d2021", "textColor": "#d4be98", "noteBkgColor": "#32302f", "noteTextColor": "#d4be98", "noteBorderColor": "#d8a657", "actorBkg": "#32302f", "actorTextColor": "#d4be98", "actorLineColor": "#d4be98", "signalColor": "#7daea3", "signalTextColor": "#d4be98", "labelBoxBkgColor": "#32302f", "labelBoxBorderColor": "#d4be98", "labelTextColor": "#d4be98", "loopTextColor": "#d4be98"}}%%
flowchart LR
    subgraph ClientSide[Cliente / Navegador]
        A[Frontend Angular]
    end
    subgraph Backend[Backend Spring Boot]
        C[API REST]
        D[Auth y Roles]
        E[Ordenes]
        F[Productos / Catalogo]
        G[Stock / Inventario]
        H[Proveedores]
        I[Reportes / Recomendaciones]
    end
    subgraph Data[Persistencia]
        N[(PostgreSQL)]
    end
    A --> C
    C --> D
    C --> E
    C --> F
    C --> G
    C --> H
    C --> I
    D --> N
    E --> N
    F --> N
    G --> N
    H --> N
    I --> N
```

## Stack tecnologico

**Backend**: Java 21, Spring Boot, JPA/Hibernate, Flyway, JUnit + Testcontainers.
**Frontend**: Angular, Signals, Tailwind/CSS, lazy loading.
**Infra**: PostgreSQL, Docker Compose, Nginx.

---

> [!seealso] Volver a [[_Index]]
