# ui-integration-tests

**Alcance:** [frontend/src/App.tsx](../../frontend/src/App.tsx) y los componentes en `frontend/src/components/dashboard/`.

**Razón:** hoy solo existen tests unitarios de funciones puras (`frontend/src/lib/financial-utils.test.ts`); no hay ningún test que monte `<App />` y verifique los tres estados posibles (loading, éxito, error). Verificado además que `frontend/package.json` **no** incluye `@testing-library/react` ni `@testing-library/jest-dom`, y que `frontend/vite.config.ts` no configura `test.environment` — es decir, el proyecto **no puede correr tests de componentes todavía**, aunque sí corre tests de funciones puras vía Vitest en Node.

**Regla (dos pasos, en orden):**
1. **Prerequisito:** agregar `@testing-library/react` y `@testing-library/jest-dom` a `devDependencies`, y configurar `test: { environment: "jsdom" }` (agregando `jsdom` como dependencia) en `frontend/vite.config.ts`.
2. **Recién entonces**, exigir al menos un test de integración por vista principal que mockee `global.fetch` y verifique loading → datos y loading → error, siguiendo el estilo de aserciones ya usado en `financial-utils.test.ts`.

**Validación:** refinado — la versión inicial de esta regla pedía directamente "agregar tests de integración de UI", como si fuera accionable de inmediato. Al revisar `package.json` y `vite.config.ts` confirmé que falta la infraestructura de testing de componentes (jsdom + Testing Library), así que la regla ahora explicita ese paso previo en vez de asumir que ya existe.
