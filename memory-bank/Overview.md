# Overview del producto

## Qué es

**Financial Metrics Dashboard**: un dashboard de métricas financieras compuesto por una API backend ([backend/app/main.py](../backend/app/main.py), FastAPI) y una SPA frontend ([frontend/src/App.tsx](../frontend/src/App.tsx), React + TypeScript) que consume esa API vía el proxy `/api` de Vite ([frontend/vite.config.ts](../frontend/vite.config.ts)).

Repositorio original: [4GeeksAcademy/bocha-ai-eng-financial-dashboard-context-project](https://github.com/4GeeksAcademy/bocha-ai-eng-financial-dashboard-context-project).

## Propósito declarado

Según [README.md](../README.md), el proyecto es un ejercicio de contexto para agentes de IA de 4Geeks Academy: se espera que un agente inspeccione el código, documente reglas y memoria de proyecto (`./.agents/rules`, `./.agents/skills`, `./memory-bank`) y las refine hasta que encajen con el flujo real del proyecto ([AGENTS.md](../AGENTS.md)). No es, en su estado actual, un producto con datos reales ni destinado a producción.

## Qué hace hoy (evidencia verificable)

El backend expone 9 endpoints sobre un dataset de movimientos financieros simulados ([backend/app/routes.py](../backend/app/routes.py)):

- `GET /health` — healthcheck.
- `GET /api/metrics` — movimientos filtrables por fecha, categoría y tipo de operación.
- `GET /api/metrics/facets` — valores disponibles para filtros (categorías, rango de fechas, etc.).
- `GET /api/metrics/summary` — resumen agregado por día/semana/mes (income, outcome, net).
- `GET /api/metrics/categories/top` — top categorías por monto.
- `GET /api/metrics/comparison` — comparación entre periodo actual y periodo anterior (delta absoluto y %).
- `GET /api/metrics/alerts` — alertas de incremento anómalo de egresos sobre una línea base histórica.
- `GET /api/metrics/b2b` y `GET /api/metrics/b2c` — movimientos segmentados por tipo de negocio.

El frontend ([frontend/src/App.tsx](../frontend/src/App.tsx)) consume `GET /api/metrics`, calcula KPIs y series mensuales con funciones puras ([frontend/src/lib/financial-utils.ts](../frontend/src/lib/financial-utils.ts)), y renderiza:

- 4 tarjetas de KPI ([frontend/src/components/dashboard/kpi-row.tsx](../frontend/src/components/dashboard/kpi-row.tsx)): Total Income, Total Outcome, Profit, Profit Margin.
- Gráfico de evolución mensual Income vs. Outcome ([frontend/src/components/dashboard/income-outcome-chart.tsx](../frontend/src/components/dashboard/income-outcome-chart.tsx)).
- Gráfico de evolución mensual del margen de ganancia ([frontend/src/components/dashboard/profit-percent-chart.tsx](../frontend/src/components/dashboard/profit-percent-chart.tsx)).
- Estados explícitos de carga y error (`loading`, `error` en `App.tsx`).

## Origen y naturaleza de los datos

No hay base de datos ni persistencia. Todos los movimientos se generan en memoria con `generate_mock_movements(seed=42)` (`backend/app/routes.py`): 12 meses × 30 registros = 360 movimientos por año, con fecha, monto, tipo de operación (`income`/`outcome`), categoría (`suppliers`, `sales`, `operational`, `administrative`, `others`) y tipo de negocio (`B2B`/`B2C`), usando una semilla fija — los mismos datos se repiten en cada arranque y en cada request.

## Alcance explícitamente fuera de este producto (hoy)

No hay, en el código actual: autenticación/autorización, persistencia real, paginación, versionado de API, ni ingestión de datos externos. Ver [Estado_actual.md](Estado_actual.md) para el detalle de gaps y prioridades, y [Stack.md](Stack.md) para el detalle tecnológico.
