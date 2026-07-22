# frontend-dto-adapter

**Alcance:** el borde de fetch de datos del frontend — [frontend/src/lib/financial-types.ts](../../frontend/src/lib/financial-types.ts) (tipo `FinancialMovement`) y `fetchFinancialData` en [frontend/src/App.tsx](../../frontend/src/App.tsx). **No** aplica a `KPIMetrics` ni `MonthlyDataPoint`, que ya están en camelCase y son producidos por `computeKPIs`/`computeMonthlyData`.

**Razón:** `FinancialMovement` en `financial-types.ts` reexpone literalmente el `snake_case` del backend (`create_date`, `operation_type`, `business_type`) como si fuera un tipo de dominio del frontend, en vez de mapearlo a una convención propia. Verificado comparando con el modelo `FinancialMovement` de `backend/app/routes.py`. Esto acopla directamente cualquier renombre futuro del backend con el frontend, sin que el compilador de TypeScript pueda avisar (la forma del DTO nunca cambia de nombre, solo su significado implícito).

**Regla:** renombrar el tipo actual a `FinancialMovementDTO` (representa la forma exacta de la respuesta HTTP) y agregar un tipo de dominio `Movement` en camelCase + una función `toMovement(dto: FinancialMovementDTO): Movement`. Los componentes de `frontend/src/components/dashboard/` deben consumir `Movement`, nunca `FinancialMovementDTO` directamente.

**Validación:** refinado — la versión inicial decía "no reutilizar tipos crudos de la API en el frontend" en términos generales, lo cual sonaba como que afectaba a todo el frontend. Al revisar `financial-utils.ts` confirmé que `computeKPIs`/`computeMonthlyData` ya producen tipos en camelCase (`KPIMetrics`, `MonthlyDataPoint`) que alimentan los componentes — el acoplamiento real está acotado al único punto donde se recibe el DTO crudo, y ahí es donde esta regla aplica.
