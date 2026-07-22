# surface-original-errors

**Alcance:** cualquier bloque `catch`/`except` del frontend o backend que hoy descarte el objeto de error recibido.

**Razón:** en `frontend/src/App.tsx` (líneas 35-38), `fetchFinancialData().catch(() => setError("mensaje genérico"))` ignora por completo el argumento de error — no queda registro de si la falla fue timeout, 500, CORS, JSON inválido, etc. Verificado leyendo el archivo.

**Regla:** todo `catch` debe capturar el error (`catch((err) => {...})`), registrarlo (`console.error(err)` en frontend / `logger.exception(...)` en backend) y solo entonces derivar, si corresponde, un mensaje amigable separado para la UI.

**Validación:** accionable de inmediato y de bajo riesgo — no cambia el comportamiento visible para el usuario (el mensaje en pantalla puede seguir siendo el mismo), solo agrega registro del error real para diagnóstico.
