# Stack tecnológico

Fuentes verificadas: [backend/requirements.txt](../backend/requirements.txt), [backend/Dockerfile](../backend/Dockerfile), [frontend/package.json](../frontend/package.json), [frontend/Dockerfile](../frontend/Dockerfile), [frontend/vite.config.ts](../frontend/vite.config.ts), [docker-compose.yml](../docker-compose.yml).

## Backend

- **Lenguaje/runtime:** Python 3.13 (`python:3.13-slim` en `backend/Dockerfile`).
- **Framework web:** FastAPI (`fastapi` en `requirements.txt`), servido con **Uvicorn** (`uvicorn[standard]`), modo `--reload` habilitado en el `CMD` del Dockerfile.
- **Validación/schemas:** Pydantic `BaseModel` (dependencia transitiva de FastAPI), usado en `backend/app/routes.py` para `FinancialMovement`, `MetricsFacets`, `MetricsSummaryItem`, `TopCategoryItem`, `MetricsComparison`, `MetricsAlert`.
- **Debug remoto:** `debugpy`, escuchando en `0.0.0.0:5678` (puerto expuesto en `backend/Dockerfile` y `docker-compose.yml`).
- **Testing:** `pytest` + `pytest-cov` (cobertura) + `httpx` (cliente usado por `fastapi.testclient.TestClient`). Tests en `backend/tests/test_routes.py`.
- **Datos:** ninguna base de datos — dataset generado en memoria (`random`, stdlib) con semilla fija.

## Frontend

- **Framework UI:** React 19.2 (`react`, `react-dom`).
- **Lenguaje:** TypeScript ~6.0 (`typescript`, `@types/react`, `@types/react-dom`, `@types/node`).
- **Build/dev server:** Vite 8 (`vite`, `@vitejs/plugin-react`), con proxy configurado en `frontend/vite.config.ts` para reenviar `/api` a `http://backend:8000` dentro de la red de Docker Compose.
- **Estilos:** TailwindCSS 4 (`tailwindcss`, `@tailwindcss/vite`, `autoprefixer`, `postcss`), utilidades `class-variance-authority`, `clsx`, `tailwind-merge`.
- **Gráficos:** Recharts 3.8 (`recharts`), usado en `income-outcome-chart.tsx` y `profit-percent-chart.tsx`.
- **Íconos:** `lucide-react`.
- **Testing:** Vitest 4 (`vitest`, `@vitest/coverage-v8`) — actualmente solo corre tests de funciones puras en Node (`frontend/src/lib/financial-utils.test.ts`); **no** tiene configurado un entorno DOM (`jsdom`/`happy-dom`) ni Testing Library, por lo que no puede correr tests de componentes React todavía (ver gap en [Estado_actual.md](Estado_actual.md)).
- **Linting:** ESLint 9 (`eslint`, `@eslint/js`, `eslint-plugin-react-hooks`, `eslint-plugin-react-refresh`, `typescript-eslint`), configurado en `frontend/eslint.config.js`.

## Infraestructura / Tooling

- **Contenedores:** Docker + Docker Compose (`docker-compose.yml`) orquesta dos servicios:
  - `frontend`: build desde `frontend/Dockerfile` (imagen `node:24-alpine`), puerto `5173`, volumen de código montado para hot-reload (`./frontend:/app`, con `/app/node_modules` como volumen anónimo para no pisar las dependencias instaladas en la imagen).
  - `backend`: build desde `backend/Dockerfile` (imagen `python:3.13-slim`), puertos `8000` (API) y `5678` (debugpy), volumen de código montado (`./backend:/app`) para reload en caliente.
- **Documentación de API:** Swagger UI autogenerado por FastAPI, servido en la raíz de la API (`http://localhost:8000`), a partir de `/openapi.json`.
- **No hay CI configurado:** no existe `.github/workflows/` en el repositorio (verificado); lint y tests se corren solo localmente (`npm run lint`, `npm run test`, `pytest`).
- **Variables de entorno:** el frontend soporta `VITE_API_BASE_URL` (opcional, ver `frontend/.env.example`) para apuntar a un backend distinto; no es necesaria en desarrollo local gracias al proxy de Vite. El backend no usa variables de entorno actualmente (ver gap de configuración en [Estado_actual.md](Estado_actual.md) y regla [cors-allowed-origins](../.agents/rules/cors-allowed-origins.md)).

## Dependencias clave (resumen)

| Capa | Dependencia | Rol |
|---|---|---|
| Backend | `fastapi` | Framework de la API |
| Backend | `uvicorn[standard]` | Servidor ASGI |
| Backend | `debugpy` | Debug remoto (puerto 5678) |
| Backend | `pytest`, `pytest-cov`, `httpx` | Testing |
| Frontend | `react`, `react-dom` | UI |
| Frontend | `vite`, `@vitejs/plugin-react` | Dev server / build, con proxy a la API |
| Frontend | `tailwindcss`, `@tailwindcss/vite` | Estilos |
| Frontend | `recharts` | Gráficos del dashboard |
| Frontend | `vitest` | Testing de funciones puras |
| Frontend | `eslint`, `typescript-eslint` | Linting/tipado estático |
