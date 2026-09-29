# VarshaNetra

Intelligent Urban Flood Prediction, Early Warning & Response System for Nagpur.

## Quick start

```bash
npm install
cp .env.example .env
npm run dev
```

Runs with `VITE_USE_MOCK_API=true` by default, so the full app — login, OTP,
dashboard, map, area details, forecast, scenarios, alerts — works with
realistic demo data and **no backend required**.

Demo login: any 10-digit number, OTP is always `123456` (also shown on
screen in demo mode).

## Connecting the real backend

1. Point `VITE_API_BASE_URL` in `.env` at your FastAPI server.
2. Set `VITE_USE_MOCK_API=false`.
3. No component code changes — every API service in `src/api/` switches
   from mock data to real HTTP calls automatically. See `API_CONTRACT.md`
   for the exact request/response shapes the backend must implement.

## Architecture

```
UI Components → Hooks (src/hooks) → API Services (src/api) → axios client → FastAPI
```

- `src/api/` — one file per domain (dashboard, zones, forecast, alerts,
  scenario, ai, auth), each with a mock/real branch controlled by
  `VITE_USE_MOCK_API`.
- `src/data/mock/` — all demo data lives here, never inline in components.
- `src/hooks/` — data-fetching hooks wrapping API services in consistent
  loading/error/data state (`useAsync`).
- `src/types/` — TypeScript interfaces mirroring the backend contract.
- `src/components/` — organized by feature (map, dashboard, risk, forecast,
  scenario, alerts, layout, common).
- `src/pages/` — one page per route.

## Core demo flow

Login → OTP → Notification permission → Dashboard → Risk Map → Khamla →
flood probability, why-at-risk factors, expected impact, AI recommendation,
traffic/bypass → Forecast → Scenario Simulator (+30% rainfall → map updates).

## Scripts

- `npm run dev` — start dev server
- `npm run build` — type-check and production build
- `npm run preview` — preview the production build
