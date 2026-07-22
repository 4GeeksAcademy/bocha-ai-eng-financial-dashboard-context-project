# mock-data-single-generation

**Alcance:** `generate_mock_movements` y los 9 endpoints de [backend/app/routes.py](../../backend/app/routes.py) que la invocan (`/api/metrics`, `/api/metrics/facets`, `/api/metrics/summary`, `/api/metrics/categories/top`, `/api/metrics/comparison`, `/api/metrics/alerts`, `/api/metrics/b2b`, `/api/metrics/b2c`).

**Razón:** cada handler llama `generate_mock_movements(seed=42)` de nuevo (líneas 255, 264, 277, 295, 311, 350, 369, 386), recalculando 360 registros en cada request en vez de generarlos una sola vez. Verificado leyendo cada endpoint.

**Regla:** envolver `generate_mock_movements` con `functools.lru_cache` (o generar el dataset una vez a nivel de módulo/dependencia de FastAPI) y reutilizarlo entre requests.

**Validación:** accionable de inmediato — todos los call-sites están identificados y usan el mismo `seed=42`, por lo que cachear no cambia el comportamiento observable actual. **Nota de alcance:** esta regla aplica solo mientras los datos sean mock deterministas; el día que se reemplacen por una fuente real (DB/API), este mismo caché dejaría de ser válido y debe removerse explícitamente, no heredarse por inercia.
