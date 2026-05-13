---
title: "Vision General de Arquitectura"
tags:
  - arquitectura
  - vision-general
  - stack
---

# Vision General de Arquitectura

> [!info] Monolito modular con separacion por features.

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
        A[Tienda Online Angular]
        B[Backoffice Angular]
    end
    subgraph Backend[Backend Spring Boot]
        C[API REST]
        D[Auth y RBAC]
        E[Pedidos]
        F[Productos]
        G[Stock]
        H[Proveedores]
        I[Ventas]
        J[Analytics]
        K[Assistant Adapter - Rule Based]
        L[Payment Adapter]
        M[Label Adapter]
    end
    subgraph Data[Persistencia]
        N[(PostgreSQL)]
    end
    subgraph External[Servicios externos / mock]
        O[Pasarela de pago]
        P[Sistema de reglas (mock MVP)]
        Q[Impresion etiquetas]
    end
    A --> C
    B --> C
    C --> D
    C --> E
    C --> F
    C --> G
    C --> H
    C --> I
    C --> J
    E --> L
    J --> N
    F --> N
    G --> N
    H --> N
    I --> N
    K --> P
    L --> O
    M --> Q
```

## Stack tecnologico

**Backend**: Java 21, Spring Boot, JPA/Hibernate, Flyway, OpenAPI, JUnit + Testcontainers.
**Frontend**: Angular, RxJS, Signals, Tailwind/CSS, lazy loading.
**Infra**: PostgreSQL, Docker Compose, Nginx (tambien para servir imagenes de producto estaticas).

## Decisiones clave

| Area | Decision | Motivo |
|---|---|---|
| Backend | Spring Boot + Java 21 | Robusto, seguridad, JPA |
| Frontend | Angular | Estructurado, lazy loading |
| BD | PostgreSQL | Relacional, transacciones |
| Despliegue | Docker Compose | Reproducible |
| Integraciones | Adaptadores | Mock hoy, integracion real despues. El asistente IA en MVP es un recomendador por reglas (sin LLM). |
| Organizacion | Por features | Evita carpetas gigantes |

---

> [!seealso] Volver a [[_Index]]
