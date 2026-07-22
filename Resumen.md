# Resumen del Proyecto — Financial Metrics Dashboard

Repositorio original: [4GeeksAcademy/bocha-ai-eng-financial-dashboard-context-project](https://github.com/4GeeksAcademy/bocha-ai-eng-financial-dashboard-context-project)

## Descripción general

Dashboard de métricas financieras compuesto por un backend en **FastAPI** (Python) y un frontend en **React + TypeScript** (Vite). El proyecto es un ejercicio de contexto para agentes de IA de 4Geeks Academy: el objetivo declarado en el README es que un agente inspeccione el código y proponga reglas/memoria de proyecto (`./.agents/rules`, `./.agents/skills`, `./memory-bank`) antes de trabajar sobre él.

## Estructura

```
├── backend/            # API FastAPI
│   ├── app/
│   │   ├── main.py     # instancia FastAPI + CORS
│   │   └── routes.py   # endpoints y lógica de métricas
│   ├── tests/           # pytest
│   ├── requirements.txt
│   └── Dockerfile
├── frontend/            # SPA React + TypeScript
│   ├── src/
│   │   ├── components/dashboard/   # KPI cards, gráficos (recharts)
│   │   ├── components/ui/          # componentes UI base (card, skeleton)
│   │   └── lib/                    # tipos, utils, datos mock
│   ├── package.json
│   └── Dockerfile
├── docker-compose.yml   # orquesta frontend (5173) + backend (8000/5678)
├── AGENTS.md            # guía para agentes de IA
├── README.md / README.es.md
```

## Backend (FastAPI)

- **Responsabilidades:**
- Generar 360 movimientos financieros anuales (12 meses x 30 registros).
- Aplicar filtros por:
	- `start_date` y `end_date`
	- `category`
	- `operation_type` (`income` / `outcome`)
	- `business_type` (`B2B` / `B2C`, según endpoint)
- Entregar agregaciones y análisis:
	- Facetas para filtros disponibles.
	- Resumen temporal por día, semana o mes.
	- Top categorías por monto.
	- Comparación entre periodo actual y periodo anterior.
	- Alertas de aumento anómalo de egresos.
- **Punto de entrada:** `backend/app/main.py` — crea la app FastAPI (`Financial Metrics API`) con CORS abierto (`allow_origins=["*"]`).
- **Datos:** no hay base de datos; `backend/app/routes.py` genera movimientos financieros simulados (`generate_mock_movements`, seed fija = 42) con fecha, monto, tipo de operación (`income`/`outcome`), categoría (`suppliers`, `sales`, `operational`, `administrative`, `others`) y tipo de negocio (`B2B`/`B2C`).
- **Endpoints principales:**
  - `GET /health` — healthcheck.
  - `GET /api/metrics` — movimientos filtrables por fecha, categoría y tipo de operación.
  - `GET /api/metrics/facets` — valores disponibles para filtros (categorías, fechas min/max, etc.).
  - `GET /api/metrics/summary` — resumen agregado por día/semana/mes (income, outcome, net).
  - `GET /api/metrics/categories/top` — top categorías por monto.
  - `GET /api/metrics/comparison` — comparación entre periodo actual y anterior (delta absoluto y %).
  - `GET /api/metrics/alerts` — alertas de incremento de gastos sobre una línea base histórica.
  - `GET /api/metrics/b2b` y `GET /api/metrics/b2c` — movimientos segmentados por tipo de negocio.
- **Testing:** `pytest` + `pytest-cov` + `httpx` (tests en `backend/tests/`).
- **Debug:** incluye `debugpy` (puerto 5678 expuesto en docker-compose) para depuración remota.

## Frontend (React + TypeScript + Vite)
- **Responsabilidades**
  - Cargar datos desde `GET /api/metrics`.
  - Calcular indicadores de negocio:
    - Ingresos totales
    - Egresos totales
    - Ganancia neta
    - Porcentaje de ganancia
  - Construir visualizaciones mensuales para:
    - Comparativa ingresos vs egresos
    - Evolución de porcentaje de rentabilidad
  - Mostrar estado de carga y manejo de error si la API falla.
- **Stack:** React 19, Vite 8, TypeScript, TailwindCSS 4, Recharts (gráficos), `class-variance-authority`/`tailwind-merge` para estilos, `lucide-react` para íconos.
- **Componentes clave** (`frontend/src/components/dashboard/`):
  - `dashboard-header.tsx` — encabezado del dashboard.
  - `kpi-card.tsx` / `kpi-row.tsx` — tarjetas de KPIs (Total Income, Total Outcome, Profit, Profit Margin).
  - `income-outcome-chart.tsx` — gráfico de evolución mensual de ingresos vs. egresos.
  - `profit-percent-chart.tsx` — gráfico de margen de ganancia mensual.
- **Consumo de API:** usa el proxy de Vite para `/api`, por lo que no requiere variables de entorno en desarrollo local (`VITE_API_BASE_URL` es opcional, ver `frontend/.env.example`).
- **Testing:** Vitest (`test`, `test:watch`, `test:coverage`) — incluye pruebas de utilidades financieras (`financial-utils.test.ts`).

## Pruebas y validación

- Backend: pruebas con `pytest` para validar endpoints, filtros y estructura de respuestas.
- Frontend: pruebas con `vitest` para validar cálculos de KPI, agregación mensual y formateadores.

Esto permite verificar tanto la lógica analítica como el contrato funcional de la API.

## Cómo ejecutarlo

```bash
docker compose up --build
```

- Frontend: http://localhost:5173
- Backend: http://localhost:8000
- Documentación interactiva de la API (Swagger UI): http://localhost:8000/docs

## Verificación realizada

Se ejecutó `docker compose up --build` con éxito:

- **Backend:** Uvicorn corriendo en `0.0.0.0:8000`, `Application startup complete`.
- **Frontend:** Vite listo en 525 ms, sirviendo en `0.0.0.0:5173`.
- **Dashboard cargado correctamente** con datos mock:
  - Total Income: $1,258,147
  - Total Outcome: $762,132
  - Profit: $496,015
  - Profit Margin: 39.4%
- Llamada `GET /api/metrics` respondida con `200 OK` a través del proxy de Vite, confirmando la integración frontend-backend.
- Swagger UI accesible en `http://localhost:8000` con todos los endpoints listados arriba disponibles para pruebas manuales.

## Notas

- Los datos son completamente simulados (mock, seed fija) — no hay persistencia real ni base de datos.
- El proyecto está pensado como caso de estudio para que un agente de IA documente reglas y memoria de trabajo propias en `./.agents`, no como una guía de arquitectura final a seguir sin revisión.
- Tiene separación clara frontend/backend.
- Incluye datos mock, visualización y analítica básica.
- Cuenta con cobertura de pruebas en ambos lados.
- Está listo para extenderse con datos reales, autenticación o nuevas métricas.