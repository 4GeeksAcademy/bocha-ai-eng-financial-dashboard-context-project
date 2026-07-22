# debug-port-opt-in

**Alcance:** `CMD` de [backend/Dockerfile](../../backend/Dockerfile) y cualquier script de arranque del backend.

**Razón:** el `CMD` actual (línea 12) siempre lanza `debugpy` escuchando en `0.0.0.0:5678`, sin condición alguna. Si este mismo Dockerfile llegara a usarse fuera de desarrollo local, quedaría un puerto de debug remoto expuesto sin autenticación.

**Regla:** el arranque de `debugpy` debe depender de una variable de entorno explícita (`DEBUG=true`). Sin ella, el contenedor debe arrancar `uvicorn` directamente, sin el wrapper de debugpy.

**Validación:** confirmado en `backend/Dockerfile`. Accionable con un cambio acotado (entrypoint condicional o dos `CMD` alternativos vía variable de build/entorno); no afecta el flujo de `docker compose up --build` ya verificado, siempre que `docker-compose.yml` siga seteando `DEBUG=true` para desarrollo.
