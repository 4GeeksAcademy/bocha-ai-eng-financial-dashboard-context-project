# backend-lint-typecheck-parity

**Alcance:** [backend/requirements.txt](../../backend/requirements.txt) y, cuando exista, la configuración de CI del repositorio.

**Razón:** el frontend tiene ESLint (`frontend/eslint.config.js`) y TypeScript (`frontend/tsconfig*.json`) configurados, con `npm run lint` disponible en `frontend/package.json`. El backend no tiene ningún linter ni type-checker en `backend/requirements.txt` (solo `fastapi`, `uvicorn[standard]`, `debugpy`, `pytest`, `pytest-cov`, `httpx`). Verificado también que el repo **no tiene CI** (no existe `.github/workflows/`).

**Regla:** agregar `ruff` (lint) y `mypy` (o `pyright`) a `backend/requirements.txt`, con configuración mínima en `pyproject.toml`. La ejecución de ambos en CI queda condicionada a que exista un pipeline — no se debe inventar un workflow de GitHub Actions como parte de esta regla si nadie lo pidió explícitamente.

**Validación:** refinado — la versión inicial mencionaba "en CI" como si ya existiera un pipeline. Confirmé que no hay `.github/workflows/` en el repo, así que la regla ahora separa claramente lo accionable ya (agregar las dependencias y su config local) de lo condicional (correrlas en CI, cuando ese pipeline exista), evitando asumir infraestructura inexistente.
