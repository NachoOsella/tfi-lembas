---
title: "Modulo Asistente Inteligente"
tags:
  - modulo
  - ia
  - inteligencia-artificial
  - recomendaciones
---

# Modulo Asistente Inteligente

> [!info] Proposito: aportar una capa diferencial que ayude al cliente a comprar mejor y al administrador a tomar decisiones comerciales.
>
> **Aclaracion para el MVP:** el asistente no utiliza IA generativa ni LLM. Se trata de un recomendador basado en reglas que filtra productos por etiquetas, stock disponible y categoria. Se denomina "inteligente" por su capacidad de interpretar texto libre y mapearlo a filtros del sistema, pero todo el procesamiento es determinista sobre datos reales de la base de datos.

## Principio tecnico

> [!important] La IA no debe inventar productos ni recomendar productos inexistentes. Trabaja sobre informacion real del sistema.

```text
Consulta del usuario -> Filtro BD (stock, categoria, tags, sucursal)
                     -> Ranking / reglas comerciales
                     -> Respuesta con productos reales y disponibles
```

## Flujo de recomendaciones

```mermaid
%%{init: {"theme": "base", "themeVariables": {"darkMode": true, "background": "#1d2021", "primaryColor": "#32302f", "primaryTextColor": "#d4be98", "primaryBorderColor": "#d4be98", "lineColor": "#d4be98", "secondaryColor": "#32302f", "tertiaryColor": "#1d2021", "textColor": "#d4be98", "noteBkgColor": "#32302f", "noteTextColor": "#d4be98", "noteBorderColor": "#d8a657", "actorBkg": "#32302f", "actorTextColor": "#d4be98", "actorLineColor": "#d4be98", "signalColor": "#7daea3", "signalTextColor": "#d4be98", "labelBoxBkgColor": "#32302f", "labelBoxBorderColor": "#d4be98", "labelTextColor": "#d4be98", "loopTextColor": "#d4be98"}}%%
flowchart TD
    A[Consulta del usuario] --> B[Clasificar intencion]
    B --> C[Consultar BD con filtros seguros]
    C --> D[Aplicar reglas comerciales]
    D --> E[Seleccionar productos candidatos]
    E --> F[Redactar respuesta]
    F --> G[Respuesta final]
```

## Dos tipos de asistente

### Para cliente

Ej: "Quiero algo dulce sin azucar", "Busco productos sin TACC". Responde con productos disponibles en la sucursal seleccionada.

### Para administrador

Ej: "Que productos reponer?", "Que promocionar?". Sugiere reposicion y promociones basadas en datos reales.

## Reglas de seguridad

- No inventar productos
- No recomendar sin stock
- No hacer diagnosticos medicos
- No exponer datos internos (costos, margenes)
- No afirmar beneficios de salud no registrados

## Prompt base interno

> Sos un asistente de compra para una dietetica. Recomenda productos solo de la lista provista por el sistema. No inventes. No recomiendes sin stock. No hagas diagnosticos. Responde breve y claro.

## Requisitos funcionales

RF-120 a RF-126 en [[05 - Requisitos/Funcionales]].

---

> [!seealso] Volver a [[_Index]]
