# Estado actual

## Features implementadas (verificadas)

- **API de métricas completa:** 9 endpoints funcionales sobre datos mock ([backend/app/routes.py](../backend/app/routes.py)) — health, listado filtrable de movimientos, facetas de filtros, resumen agregado (día/semana/mes), top categorías, comparación de periodos, alertas de egresos anómalos, y segmentación B2B/B2C.
- **Dashboard funcional:** carga datos desde el backend, calcula KPIs (Total Income, Total Outcome, Profit, Profit Margin) y series mensuales, y los renderiza en tarjetas + 2 gráficos (Recharts), con estados de loading y error visibles en la UI ([frontend/src/App.tsx](../frontend/src/App.tsx)).
- **Entorno de desarrollo reproducible:** `docker compose up --build` levanta frontend (5173) y backend (8000/5678) con hot-reload en ambos, verificado funcionando end-to-end (dashboard consumiendo datos mock servidos por la API vía proxy de Vite, `GET /api/metrics` → `200 OK`).
- **Documentación interactiva de la API:** Swagger UI autogenerado, accesible en `http://localhost:8000/docs`.
- **Cobertura de tests en ambas capas:** `pytest` sobre endpoints y funciones de dominio del backend ([backend/tests/test_routes.py](../backend/tests/test_routes.py)); `vitest` sobre las funciones puras de cálculo del frontend ([frontend/src/lib/financial-utils.test.ts](../frontend/src/lib/financial-utils.test.ts)).
- **Base de documentación de contexto para agentes:** `Resumen.md`, `Analisis-Codigo.md` y `.agents/rules/*.md` ya creados en fases previas del proyecto.

## Gaps conocidos

Detallados y validados contra el código en [Analisis-Codigo.md](../Analisis-Codigo.md) y en las reglas de `.agents/rules/`:

- **Seguridad:** CORS configurado como `allow_origins=["*"]` + `allow_credentials=True` (`backend/app/main.py`) — permite en la práctica cualquier origen con credenciales. Debugger remoto (`debugpy`) siempre activo en el `CMD` del backend, sin condicionarlo por entorno. → [cors-allowed-origins](../.agents/rules/cors-allowed-origins.md), [debug-port-opt-in](../.agents/rules/debug-port-opt-in.md).
- **Persistencia/performance:** no hay base de datos; los 360 movimientos mock se regeneran en cada request en vez de cachearse. → [mock-data-single-generation](../.agents/rules/mock-data-single-generation.md).
- **Estructura del backend:** `routes.py` mezcla modelos, lógica de dominio y handlers HTTP en un solo archivo de 391 líneas. → [routes-module-decomposition](../.agents/rules/routes-module-decomposition.md).
- **Contrato API/frontend:** el tipo `FinancialMovement` del frontend replica el `snake_case` del backend sin capa adaptadora; no hay versionado de API (`/api/v1/...`). → [frontend-dto-adapter](../.agents/rules/frontend-dto-adapter.md), [api-versioning-new-endpoints](../.agents/rules/api-versioning-new-endpoints.md), [api-field-documentation](../.agents/rules/api-field-documentation.md).
- **Manejo de errores:** el `catch` en `App.tsx` descarta el error real y solo muestra un mensaje genérico; `get_metrics_comparison` no valida que `end_date >= start_date`. → [surface-original-errors](../.agents/rules/surface-original-errors.md), [date-range-validation](../.agents/rules/date-range-validation.md).
- **Testing de UI:** no hay tests de integración de componentes React; falta la infraestructura misma para correrlos (sin `@testing-library/react` ni entorno `jsdom` en Vitest). → [ui-integration-tests](../.agents/rules/ui-integration-tests.md).
- **DX backend:** sin linter ni type-checker en el backend (`ruff`/`mypy`), a diferencia del frontend que sí tiene ESLint; tampoco existe CI (`.github/workflows/`) en el repo. → [backend-lint-typecheck-parity](../.agents/rules/backend-lint-typecheck-parity.md).
- **Datos:** 100% mock con semilla fija (`seed=42`) — no hay ingestión de datos reales, ni autenticación/autorización en ningún endpoint.

## Siguientes prioridades (propuestas)

Ordenadas por impacto/riesgo vs. esfuerzo, en base a los gaps de arriba:

1. **Cerrar riesgos de seguridad ya identificados** (bajo esfuerzo, alto impacto): aplicar `cors-allowed-origins` y `debug-port-opt-in` antes de cualquier despliegue fuera de `docker-compose.yml` local.
2. **Corrección de correctness de bajo riesgo:** `date-range-validation` y `surface-original-errors` — cambios acotados, no rompen tests existentes.
3. **Habilitar testing de componentes:** instalar `@testing-library/react` + `jsdom` (prerequisito de `ui-integration-tests`) para poder cubrir los estados loading/error de la UI, hoy no verificados por tests.
4. **Higiene de backend:** agregar `ruff`/`mypy` (`backend-lint-typecheck-parity`) y documentar campos de los modelos Pydantic (`api-field-documentation`).
5. **Refactors estructurales de mayor esfuerzo:** `routes-module-decomposition` y `frontend-dto-adapter` — conviene abordarlos antes de agregar features nuevas sobre el estado actual, para no seguir acumulando deuda en el mismo archivo/tipo.
6. **Evaluar necesidad real de datos reales/persistencia:** hoy no hay ningún endpoint de escritura ni fuente de datos externa; antes de construir sobre `mock-data-single-generation`, decidir si el próximo paso del proyecto es conectar una fuente real (lo cual invalidaría ese caching y requeriría diseño de persistencia aparte).
