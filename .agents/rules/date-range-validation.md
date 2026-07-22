# date-range-validation

**Alcance:** `get_metrics_comparison` en [backend/app/routes.py](../../backend/app/routes.py) (líneas 305-339); aplica también a cualquier endpoint futuro que reciba `start_date`/`end_date` como parámetros de query.

**Razón:** el endpoint no valida que `end_date >= start_date`. Con fechas invertidas, `duration = end_date - start_date` es negativa, y el período "anterior" calculado (`previous_start`/`previous_end`) pierde sentido — la API igual responde `200` con datos incorrectos en silencio. Verificado leyendo la función.

**Regla:** agregar una validación explícita al inicio del handler (o un `model_validator` si se migra a un modelo de query) que devuelva `400 Bad Request` cuando `end_date < start_date`.

**Validación:** accionable de inmediato. Confirmado que no rompe los tests existentes: `test_metrics_comparison_returns_delta_fields` en `backend/tests/test_routes.py` ya usa un rango válido (`2025-03-01` a `2025-03-31`).
