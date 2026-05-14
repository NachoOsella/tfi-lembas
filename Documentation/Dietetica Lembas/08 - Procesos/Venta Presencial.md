---
title: "Venta Presencial (POS)"
tags:
  - procesos
  - venta-presencial
  - pos
  - caja
---

# Proceso: Venta Presencial con Caja Operativa

> [!info] Empleado abre caja, registra ventas con distintos medios de pago, cierra caja con arqueo de efectivo.

---

## Diagrama de secuencia

```mermaid
%%{init: {"theme": "base", "themeVariables": {"darkMode": true, "background": "#1d2021", "primaryColor": "#32302f", "primaryTextColor": "#d4be98", "primaryBorderColor": "#d4be98", "lineColor": "#d4be98", "secondaryColor": "#32302f", "tertiaryColor": "#1d2021", "textColor": "#d4be98", "noteBkgColor": "#32302f", "noteTextColor": "#d4be98", "noteBorderColor": "#d8a657", "actorBkg": "#32302f", "actorTextColor": "#d4be98", "actorLineColor": "#d4be98", "signalColor": "#7daea3", "signalTextColor": "#d4be98", "labelBoxBkgColor": "#32302f", "labelBoxBorderColor": "#d4be98", "labelTextColor": "#d4be98", "loopTextColor": "#d4be98"}}%%
sequenceDiagram
    actor E as Empleado
    participant FE as Frontend
    participant BE as Backend
    participant DB as PostgreSQL

    Note over E,DB: 1. APERTURA DE CAJA
    E->>FE: Ingresa monto inicial de efectivo
    FE->>BE: POST /api/admin/cash-sessions/open
    BE->>DB: Verificar que no haya otra caja abierta
    BE->>DB: INSERT cash_session (status=OPEN, openingCashAmount)
    BE-->>FE: Caja abierta

    Note over E,DB: 2. VENTA PRESENCIAL
    E->>FE: Escanea productos
    FE->>BE: GET /api/store/products?barcode=...
    BE-->>FE: Producto con precio y stock
    FE->>FE: Arma ticket

    Note over E,DB: 3. COBRAR
    Cl-->>E: Paga con QR
    E->>FE: Metodo: QR, confirma venta
    FE->>BE: POST /api/admin/pos/sales
    BE->>BE: @Transactional
    BE->>DB: Validar caja abierta
    BE->>DB: Validar stock (FOR UPDATE)
    BE->>DB: UPDATE stock_lots (descontar FEFO)
    BE->>DB: INSERT stock_movement (type=POS_SALE)
    BE->>DB: INSERT order (type=POS, status=PAID)
    BE->>DB: INSERT order_items (snapshots)
    BE->>DB: INSERT payment (cash_session_id, provider=MANUAL, method=QR, status=APPROVED)
    BE-->>FE: Venta completada

    Note over E,DB: 4. CIERRE DE CAJA
    E->>FE: Solicita cierre
    FE->>BE: GET /api/admin/cash-sessions/current (ver resumen)
    BE-->>FE: Totales por medio de pago, efectivo esperado
    E->>FE: Ingresa efectivo contado y explica diferencia si existe
    FE->>BE: POST /api/admin/cash-sessions/{id}/close
    BE->>BE: Calcular expectedCashAmount
    BE->>DB: Verificar si hay diferencia
    Note over BE,DB: Si difference != 0 y no hay reason → error
    BE->>DB: UPDATE cash_session (status=CLOSED, countedCashAmount, cashDifferenceReason)
    BE-->>FE: Caja cerrada con detalle completo
```

## Logica de cierre de caja

```text
expectedCashAmount = openingCashAmount
                   + SUM(payments WHERE method=CASH AND status=APPROVED)
                   + SUM(cash_movements WHERE type=CASH_IN AND method=CASH)
                   - SUM(cash_movements WHERE type=CASH_OUT AND method=CASH)

cashDifference = countedCashAmount - expectedCashAmount

Si cashDifference != 0:
  → cash_difference_reason es OBLIGATORIO
  → La caja se cierra igual, la diferencia queda auditada
```

## Totales informativos del cierre (por metodo de pago)

```text
- CASH:        $X (afecta efectivo esperado)
- QR:          $Y (no afecta efectivo, solo informativo)
- TRANSFER:    $Z (no afecta efectivo)
- DEBIT_CARD:  $W (no afecta efectivo)
- CREDIT_CARD: $V (no afecta efectivo)
```

---

> [!seealso] Notas relacionadas
> - [[02 - Modulos/Backoffice]]
> - [[03 - Dominio/Reglas de Stock]]
> - Volver a [[08 - Procesos/_Index]]
