# Análisis de Código — Financial Metrics Dashboard

Revisión del código del backend (FastAPI) y frontend (React + TypeScript) identificando buenas prácticas, malas prácticas/riesgos, agrupados por categoría.

## Buenas prácticas

### Arquitectura
1. **Separación clara de responsabilidades:** backend (API/datos) y frontend (presentación) son proyectos independientes, cada uno con su propio Dockerfile, orquestados vía `docker-compose.yml`.
2. **Lógica de negocio expresada como funciones puras y sin efectos secundarios:** `filter_movements`, `summarize_movements`, `build_top_categories`, `detect_outcome_alerts` en [backend/app/routes.py](backend/app/routes.py) y `computeKPIs`, `computeMonthlyData` en [frontend/src/lib/financial-utils.ts](frontend/src/lib/financial-utils.ts). Esto las hace triviales de testear y reutilizar.

### Tipado y contrato de datos
3. **Tipado fuerte en ambos extremos:** Pydantic + `Literal` en el backend ([backend/app/routes.py](backend/app/routes.py)) y TypeScript en el frontend ([frontend/src/lib/financial-types.ts](frontend/src/lib/financial-types.ts)), lo que documenta el contrato de la API y habilita el Swagger autogenerado.

### Testing
4. **Cobertura de tests en ambas capas:** `pytest` sobre endpoints y funciones de dominio ([backend/tests/test_routes.py](backend/tests/test_routes.py)), `vitest` sobre las utilidades financieras del frontend — separando tests de lógica pura de tests de integración HTTP.

### Developer Experience (DX)
5. **Buen setup de entorno de desarrollo:** proxy de Vite hacia `/api` (evita configurar CORS/env vars en local), volúmenes montados en `docker-compose.yml` para hot-reload, y `debugpy` ya cableado en el backend para debug remoto.
6. **Manejo explícito de estados de carga/error en la UI:** `loading` y `error` en [frontend/src/App.tsx](frontend/src/App.tsx) con `aria-label` para accesibilidad.

## Malas prácticas / riesgos

### Seguridad
1. **CORS demasiado permisivo:** `allow_origins=["*"]` combinado con `allow_credentials=True` en [backend/app/main.py](backend/app/main.py) (líneas 7-13). Starlette resuelve esto reflejando el `Origin` de cada request, lo que en la práctica permite *cualquier origen con credenciales* — el peor caso posible de configuración CORS.
2. **Debugger remoto siempre activo:** el `CMD` del [backend/Dockerfile](backend/Dockerfile) (línea 12) levanta `debugpy` escuchando en `0.0.0.0:5678` en toda ejecución, sin diferenciar dev de otros entornos. Si este mismo Dockerfile se usara para desplegar, quedaría un puerto de debug expuesto sin autenticación.

### Arquitectura
3. **Regeneración de datos en cada request:** cada endpoint llama `generate_mock_movements(seed=42)` de nuevo (ej. [backend/app/routes.py:255,264,277](backend/app/routes.py)) — recalcula 360 registros por cada llamada en vez de cachear o inyectar los datos una vez. Funciona con mocks, pero es un patrón que no escala si se reemplaza por datos reales.
4. **Archivo "todo-en-uno":** [backend/app/routes.py](backend/app/routes.py) mezcla modelos Pydantic, lógica de dominio y handlers HTTP en 392 líneas sin separación (`models.py`, `services.py`, `routers.py`). Se vuelve difícil de mantener a medida que crezca.

### Naming / contrato API-Frontend
5. **Acoplamiento naming backend↔frontend:** el frontend replica literalmente el `snake_case` del backend (`create_date`, `operation_type`) en sus tipos TS ([frontend/src/lib/financial-types.ts](frontend/src/lib/financial-types.ts)) en lugar de mapear a `camelCase` idiomático. No hay capa adaptadora — un cambio de nombre en el backend rompe el frontend directamente, sin aviso de tipos.

### Manejo de errores
6. **Error real descartado en el frontend:** en [frontend/src/App.tsx](frontend/src/App.tsx) (líneas 35-38) el `catch` ignora el error capturado y siempre muestra un mensaje genérico — dificulta diagnosticar fallos reales (timeout, 500, CORS, etc.) en producción.
7. **Sin validación de rangos de fecha:** `get_metrics_comparison` ([backend/app/routes.py](backend/app/routes.py), líneas 305-339) no valida que `end_date >= start_date`; con fechas invertidas la duración da negativa y el período "anterior" calculado no tiene sentido, sin que la API lo rechace.

### Testing / DX
8. **Sin tests de integración de UI:** solo hay tests unitarios de funciones puras en el frontend; no hay tests de `App.tsx` para los estados de loading/error, ni tests de contrato (schema) entre backend y frontend — riesgo directo dado el punto 5.
9. **Asimetría de calidad de herramientas:** el frontend tiene ESLint configurado; el backend no tiene linter/formatter ni type-checker (`ruff`, `mypy`, `black`) configurado en `requirements.txt` ni en CI.

## Resumen

| Categoría | Buenas prácticas | Riesgos |
|---|---|---|
| Arquitectura | Separación FE/BE, funciones puras | Regeneración de datos por request, archivo monolítico `routes.py` |
| Tipado/contrato | Pydantic + TypeScript en ambos lados | Acoplamiento naming snake_case sin capa adaptadora |
| Testing | Cobertura pytest + vitest en lógica pura | Sin tests de integración UI ni de contrato API |
| DX | Proxy Vite, hot-reload, debugpy | Sin linter/type-checker en backend |
| Seguridad | — | CORS `*` + credentials, debugger expuesto por defecto |
| Manejo de errores | Estados loading/error explícitos en UI | Error real descartado, sin validación de rango de fechas |

Ver las reglas propuestas para mitigar estos riesgos en [.agents/rules/](.agents/rules/README.md).
