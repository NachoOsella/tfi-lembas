---
title: "Estrategia de Despliegue"
tags:
  - referencia
  - despliegue
  - docker
---

# Estrategia de Despliegue (MVP)

---

## Ambiente local (Docker Compose)

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

| Servicio | Componente |
|---|---|
| VPS (DigitalOcean, Hetzner) | Backend + Base de datos |
| Vercel / Netlify | Frontend |
| Nginx | Reverse proxy y archivos estaticos |

---

## Variables de entorno

```text
DATABASE_URL
DATABASE_USER
DATABASE_PASSWORD
JWT_SECRET
APP_PUBLIC_URL
```

---

> Volver a [[_Index]]
