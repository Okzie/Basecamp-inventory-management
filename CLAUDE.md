# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Factory Inventory Management System demo: Vue 3 + Vite frontend, FastAPI backend, in-memory mock data loaded from JSON (no database, no auth).

Nested `CLAUDE.md` files in `client/` and `server/` contain deeper conventions for each side — read them when working inside those directories.

## Critical Tool Usage Rules

### Subagents (via Task tool)
- **vue-expert**: **MANDATORY** for ANY creation or significant modification of a `.vue` file. Also for Vue 3 reactivity, state management, and styling work.
- **code-reviewer**: After writing significant code.
- **Explore**: For codebase searches / understanding component relationships.
- **general-purpose**: Complex multi-step tasks not covered above.

### Skills
- **backend-api-test**: Use when writing or modifying tests in `tests/backend/`.
- **saas-redesign**: Use when the user asks to modernize the UI, add a sidebar layout, introduce a design system, or apply a SaaS-style visual refresh to the Vue 3 client.

### MCP Tools (configured in `.mcp.json`)
- **GitHub MCP** (`mcp__github__*`): Use for ALL GitHub operations (issues, PRs, remote branches). Local branches: use `git checkout -b` instead.
- **Playwright MCP** (`mcp__playwright__*`): Use for browser testing. Targets `http://localhost:3000` (frontend) and `http://localhost:8001` (API).

## Commands

### Run the full stack (recommended)
```bash
./scripts/start.sh    # Boots backend + frontend, logs to /tmp/inventory-*.log
./scripts/stop.sh     # Stops both
```
Scripts are macOS/Linux only — on Windows use the manual commands below.

### Backend (FastAPI, port 8001)
```bash
cd server
uv venv && uv sync           # First-time setup
uv run python main.py        # Run dev server
```
API docs auto-served at `http://localhost:8001/docs`.

### Frontend (Vue 3 + Vite, port 3000)
```bash
cd client
npm install                  # First-time setup
npm run dev                  # Dev server
npm run build                # Production build → client/dist/
npm run preview              # Preview production build
```

### Tests (pytest, FastAPI TestClient, ~51 tests)
```bash
cd tests
uv run pytest -v                                                              # All tests
uv run pytest backend/test_inventory.py -v                                    # One file
uv run pytest backend/test_inventory.py::TestInventoryEndpoints -v            # One class
uv run pytest backend/test_inventory.py::TestInventoryEndpoints::test_get_all_inventory -v   # One test
uv run pytest --cov=../server --cov-report=html                               # With coverage
```
`pytest.ini` lives in `tests/` — run pytest from that directory.

## Architecture

### Big picture data flow
```
Vue view (e.g. Dashboard.vue)
   └─ useFilters() composable (shared filter refs)
        └─ client/src/api.js (axios, builds query string, drops 'all' values)
             └─ FastAPI endpoint in server/main.py
                  └─ apply_filters() / filter_by_month() helpers
                       └─ in-memory lists from server/mock_data.py (JSON-loaded at startup)
                            └─ Pydantic response_model validation → JSON → client
```

### Filter system
Four global filters apply across the app via query params: `warehouse`, `category`, `status`, `month`. They are:
- Defined once in `client/src/composables/useFilters.js` (shared module-scoped refs).
- Bound by `client/src/components/FilterBar.vue`.
- Forwarded by every `api.js` method that supports them.
- Stripped server-side when value is `'all'` (see `apply_filters` in `server/main.py`).
- `month` accepts either `YYYY-MM` or `Q1-2025`…`Q4-2025` (see `QUARTER_MAP` in `server/main.py`). Inventory endpoints don't accept `month` (no time dimension).

### Reactivity convention (Vue)
Raw API data lives in `ref`s (e.g. `allOrders`, `inventoryItems`). Anything derived (filtered/grouped/aggregated) lives in `computed`. Do not duplicate filter logic on the client when the server already filters via query params.

### Data lifecycle
- All data is loaded from `server/data/*.json` into memory at FastAPI startup via `server/mock_data.py`.
- Mutations to in-memory data do NOT persist — restarting the server reloads from JSON.
- When changing JSON schema, update the matching Pydantic model in `server/main.py`.

### Frontend extras
- **Router**: `client/src/main.js` defines routes (`/`, `/inventory`, `/orders`, `/demand`, `/spending`, `/reports`).
- **i18n**: `client/src/composables/useI18n.js` + `client/src/locales/{en,ja}.js` + `LanguageSwitcher.vue`.
- **Auth (mock)**: `client/src/composables/useAuth.js` — demo only, no real auth.
- **Currency utils**: `client/src/utils/currency.js`.

## API Endpoints (`server/main.py`)

| Endpoint | Filters | Notes |
|---|---|---|
| `GET /api/inventory` | warehouse, category | |
| `GET /api/inventory/{id}` | — | 404 if missing |
| `GET /api/orders` | warehouse, category, status, month | |
| `GET /api/orders/{id}` | — | 404 if missing |
| `GET /api/demand` | — | |
| `GET /api/backlog` | — | Augments items with `has_purchase_order` flag |
| `GET /api/dashboard/summary` | warehouse, category, status, month | |
| `GET /api/spending/summary` | — | |
| `GET /api/spending/monthly` | — | |
| `GET /api/spending/categories` | — | |
| `GET /api/spending/transactions` | — | |
| `GET /api/reports/quarterly` | — | Computed from orders |
| `GET /api/reports/monthly-trends` | — | Computed from orders |

Note: `client/src/api.js` includes calls for `/api/tasks` and `/api/purchase-orders` — these endpoints are not yet implemented in `server/main.py`. Add the Pydantic model + route together when wiring them up.

## Common Gotchas
1. **v-for keys**: never use array index — use `sku`, `id`, `month`, etc. Wrong keys cause Vue to reuse DOM nodes incorrectly when filters change.
2. **Date parsing**: validate with `!isNaN(date.getTime())` before calling `.getMonth()`.
3. **Pydantic drift**: changing the shape of any `server/data/*.json` file requires updating the matching `BaseModel` in `server/main.py` or responses 500.
4. **Inventory has no time dimension** — don't pass `month` to `/api/inventory`.
5. **Demo revenue goals**: $800K/month per single month, $9.6M YTD across all months. Used in dashboard targets.
6. **CORS** is wide open (`allow_origins=["*"]`) — intentional for the demo.

## Design System
- Palette: slate/gray (`#0f172a`, `#64748b`, `#e2e8f0`).
- Status colors: green / blue / yellow / red.
- Charts: hand-rolled SVG (no chart library); layouts use CSS Grid.
- **No emojis in UI.**

## File Map
- Views: `client/src/views/*.vue` (Dashboard, Inventory, Orders, Demand, Spending, Reports, Backlog)
- Components: `client/src/components/*.vue`
- Composables: `client/src/composables/{useFilters,useAuth,useI18n}.js`
- API client: `client/src/api.js`
- Global styles: `client/src/App.vue`
- Backend entrypoint + endpoints + models: `server/main.py`
- Data loader: `server/mock_data.py`
- JSON fixtures: `server/data/*.json`
- Tests: `tests/backend/test_*.py`, fixtures in `tests/backend/conftest.py`
