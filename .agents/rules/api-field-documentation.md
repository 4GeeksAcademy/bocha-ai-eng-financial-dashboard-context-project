# api-field-documentation

**Alcance:** todos los modelos Pydantic y decoradores de endpoint en [backend/app/routes.py](../../backend/app/routes.py).

**Razón:** los modelos (`FinancialMovement`, `MetricsSummaryItem`, `MetricsComparison`, etc.) no tienen `Field(description=...)` ni los endpoints tienen `summary`/`description`. El Swagger autogenerado (verificado en `http://localhost:8000`) hoy solo expone nombres de campos y tipos, sin explicar semántica (ej. qué significa `increase_ratio` en `MetricsAlert`, o qué convención de signo tiene `delta_pct` en `MetricsComparison`).

**Regla:** todo campo Pydantic nuevo o modificado debe incluir `Field(description=...)`; todo endpoint nuevo debe incluir `summary` en el decorador `@router.get(...)`.

**Validación:** accionable de inmediato, sin riesgo — es un cambio aditivo que no altera el contrato JSON ni rompe los tests existentes en `backend/tests/test_routes.py`.
