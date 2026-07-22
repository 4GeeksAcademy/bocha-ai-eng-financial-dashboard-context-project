# cors-allowed-origins

**Alcance:** `CORSMiddleware` en [backend/app/main.py](../../backend/app/main.py), y cualquier middleware CORS que se agregue en el futuro.

**Razón:** el código actual configura `allow_origins=["*"]` junto con `allow_credentials=True` (backend/app/main.py, líneas 7-13). Starlette resuelve esta combinación reflejando el header `Origin` de cada request en vez de rechazarla, lo que en la práctica permite *cualquier origen con credenciales* — el peor caso posible de política CORS.

**Regla:** nunca combinar `allow_origins=["*"]` con `allow_credentials=True`. Definir la lista de orígenes permitidos vía variable de entorno `ALLOWED_ORIGINS` (coma-separada), con default `["http://localhost:5173"]` para el entorno de `docker-compose.yml`.

**Validación:** confirmado leyendo `backend/app/main.py`. Es accionable de inmediato: no requiere cambios de infraestructura, solo reemplazar el literal `["*"]` por una lista parseada desde env var.
