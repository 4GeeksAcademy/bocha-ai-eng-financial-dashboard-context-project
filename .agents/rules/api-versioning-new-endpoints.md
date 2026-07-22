# api-versioning-new-endpoints

**Alcance:** nuevos grupos de endpoints que se agreguen al backend. **No** aplica retroactivamente a los 9 endpoints actuales bajo `/api/...`.

**Razón:** ningún endpoint actual tiene versión en el path (`/api/metrics`, `/api/metrics/summary`, etc.), y el frontend los consume vía el proxy de Vite configurado exactamente para `/api` ([frontend/vite.config.ts](../../frontend/vite.config.ts)). Forzar un prefijo `/api/v1/...` retroactivo hoy rompería ese proxy y el fetch en `frontend/src/App.tsx` sin ningún beneficio inmediato — sería un cambio desconectado del flujo real del proyecto.

**Regla:** cualquier **grupo nuevo** de endpoints (no una extensión de uno existente) debe nacer bajo un prefijo versionado explícito (`/api/v1/...`). Los endpoints actuales se tratan como el `v1` implícito y solo se migran al prefijo explícito si se hace, en el mismo cambio, la actualización correspondiente del proxy de Vite y del fetch del frontend.

**Validación:** refinado — la versión inicial de esta regla proponía renombrar todo `/api/...` a `/api/v1/...` de forma genérica, sin considerar que el frontend depende del path exacto vía el proxy de Vite. Ahora el alcance está acotado para no generar una tarea de refactor no solicitada ni romper el contrato ya verificado en funcionamiento.
