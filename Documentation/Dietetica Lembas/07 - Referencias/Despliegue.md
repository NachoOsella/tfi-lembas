---
title: "Estrategia de Despliegue"
tags:
  - referencia
  - despliegue
  - docker
  - devops
---

# Estrategia de Despliegue

> [!info] Estrategia para desplegar el sistema en entornos local y de demo.

---

## Ambiente local

**Docker Compose** con:

- PostgreSQL (base de datos)
- Backend Spring Boot (API REST)
- Frontend Angular (servido por Nginx o Angular)
- Servicio mock de pagos (opcional)

```yaml
services:
  postgres:
    image: postgres:16
    environment:
      POSTGRES_DB: lembas
      POSTGRES_USER: lembas
      POSTGRES_PASSWORD: secret

  backend:
    build: ./backend
    environment:
      DATABASE_URL: jdbc:postgresql://postgres:5432/lembas
      PAYMENT_PROVIDER_MODE: mock
      AI_PROVIDER_MODE: mock
    ports:
      - "8080:8080"
    depends_on:
      - postgres

  frontend:
    build: ./frontend
    ports:
      - "4200:80"
    depends_on:
      - backend
```

---

## Ambiente de demo

Opciones de hosting:

| Servicio | Componente | Costo |
|---|---|---|
| VPS (DigitalOcean, Hetzner) | Backend + Base de datos | Bajo |
| Railway / Render / Fly.io | Backend | Gratis/bajo |
| Vercel / Netlify | Frontend | Gratuito |
| Caddy / Nginx | Reverse proxy | Gratuito |

### Variables de entorno

```text
DATABASE_URL
DATABASE_USER
DATABASE_PASSWORD
JWT_SECRET
PAYMENT_PROVIDER_MODE=mock|mercadopago
AI_PROVIDER_MODE=mock|llm
AI_API_KEY
APP_PUBLIC_URL
```

---

## Observabilidad basica

Para tesis puede incluirse:

- Logs estructurados (JSON)
- Health check endpoint (`/actuator/health`)
- Metricas basicas con Spring Boot Actuator
- Documentacion de API con Swagger/OpenAPI

---

> [!seealso] Notas relacionadas
> - Volver a [[_Index]]
