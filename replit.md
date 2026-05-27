# CryptoBot Signal Monitor

A crypto trading bot with signal alerts and a live web dashboard. The bot analyzes Binance market data using EMA crossover + RSI strategy and generates BUY/SELL/HOLD signals — no automatic order placement.

## Run & Operate

- `pnpm --filter @workspace/api-server run dev` — run the API server (port 8080)
- `pnpm --filter @workspace/trading-dashboard run dev` — run the dashboard frontend
- `pnpm run typecheck` — full typecheck across all packages
- `pnpm run build` — typecheck + build all packages
- `pnpm --filter @workspace/api-spec run codegen` — regenerate API hooks and Zod schemas from the OpenAPI spec
- `pnpm --filter @workspace/db run push` — push DB schema changes (dev only)
- Required env: `DATABASE_URL` — Postgres connection string (auto-provisioned)

### Python Bot

```bash
cd artifacts/trading-bot
pip install -r requirements.txt
python src/bot.py
```

## Stack

- pnpm workspaces, Node.js 24, TypeScript 5.9
- API: Express 5
- Frontend: React + Vite, TanStack Query, wouter, recharts, shadcn/ui
- DB: PostgreSQL + Drizzle ORM
- Validation: Zod (`zod/v4`), `drizzle-zod`
- API codegen: Orval (from OpenAPI spec)
- Build: esbuild (CJS bundle)
- Python bot: psycopg2, requests

## Where things live

- `lib/api-spec/openapi.yaml` — API contract (source of truth)
- `lib/db/src/schema/` — DB schema (`signals.ts`, `bot_config.ts`)
- `artifacts/api-server/src/routes/` — route handlers (`bot.ts`, `signals.ts`, `market.ts`)
- `artifacts/api-server/src/lib/binance.ts` — Binance API client + technical indicators
- `artifacts/trading-dashboard/src/` — React frontend
- `artifacts/trading-bot/src/bot.py` — Python signal bot

## Architecture decisions

- Signal-only mode: bot generates alerts but never places orders — safe by design
- Binance geo-fallback: all mirrors tried in order; falls back to realistic mock data when geo-restricted
- Bot config in DB: Python bot reads config every scan, so dashboard changes take effect immediately without restart
- EMA + RSI strategy: fast EMA(9)/slow EMA(21) crossover + RSI(14) confirmation + RSI overbought/oversold extremes

## Product

- **Dashboard** (`/`) — live price, latest signals, market summary, bot start/stop
- **Signals** (`/signals`) — full history with symbol + type filters
- **Chart** (`/chart`) — price area chart with EMA overlays + RSI sub-chart
- **Config** (`/config`) — change symbol, interval, EMA/RSI parameters

## Gotchas

- Binance returns 451 in some environments (geo-restricted). The server automatically falls back to realistic mock prices — the app will work but prices won't be real. Deploy to a non-restricted region to get live data.
- Always run codegen after any OpenAPI spec change: `pnpm --filter @workspace/api-spec run codegen`
- The Python bot and Node API server are independent — bot writes signals directly to the DB; the API server reads them.
