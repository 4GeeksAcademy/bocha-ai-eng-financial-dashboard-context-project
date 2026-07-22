# routes-module-decomposition

**Alcance:** [backend/app/routes.py](../../backend/app/routes.py) (391 líneas) y cualquier módulo de rutas que se agregue en el backend.

**Razón:** el archivo mezcla 8 modelos Pydantic (`FinancialMovement`, `MetricsFacets`, `MetricsSummaryItem`, `TopCategoryItem`, `MetricsComparison`, `MetricsAlert`, etc.), ~10 funciones de lógica de dominio (`filter_movements`, `summarize_movements`, `build_top_categories`, `detect_outcome_alerts`, ...) y 9 handlers HTTP, todo en un único archivo.

**Regla:** cuando un módulo de rutas defina **modelos + lógica de negocio + handlers HTTP** simultáneamente, dividirlo en `models.py` (schemas Pydantic), `services.py` (funciones puras de dominio) y `routers.py` (handlers HTTP finos que solo orquestan). El disparador es la mezcla de responsabilidades, no un conteo de líneas.

**Validación:** refinado — la versión inicial de esta regla usaba un umbral arbitrario ("150-200 líneas"), que es ambiguo y fácil de discutir caso a caso. El criterio ahora es estructural y verificable objetivamente (¿el archivo tiene las tres cosas a la vez?), y aplica hoy mismo a `routes.py`, que ya cumple las tres condiciones.
