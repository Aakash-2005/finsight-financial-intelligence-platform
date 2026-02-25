## FinSight — A Unified Financial Intelligence & Decision-Support Platform

FinSight is a startup-grade FinTech MVP built for hackathon demos: **JWT auth**, a **VisualPe-inspired smart financial dashboard**, **budget & expense tracking**, **investment insights**, **AI-style rule-based insights**, **smart alerts**, and an in-browser **Android simulator** preview.

**Inspired by VisualPe** — Enhanced with multi-section financial views, portfolio breakdowns, assets/liabilities tracking, UPI payments visualization, and polished card-based layouts.

Built by **Team focus_4**.

### Tech stack

- **Frontend**: Next.js (App Router) + TypeScript + Tailwind v4 + shadcn/ui + Framer Motion + Recharts
- **Backend**: Next.js Route Handlers (Node.js runtime)
- **DB**: MongoDB (Mongoose)
- **Auth**: JWT (HttpOnly cookie) + bcrypt hashing
- **Extras**: Dark/Light mode, PDF export, goal tracker, bank linking mock, biometric mock, fraud simulation alerts

### Local setup (recommended)

1) Install dependencies

```bash
npm install
```

2) Configure environment variables

Copy `.env.example` → `.env.local` and set:

- `MONGODB_URI`
- `JWT_SECRET`

3) Run MongoDB

- **Option A (local MongoDB)**: ensure MongoDB is running and `MONGODB_URI` points to it.
- **Option B (Docker MongoDB)**:

```bash
docker compose up -d mongo
```

4) Start the app

```bash
npm run dev
```

Open `http://localhost:3000`.

**If you get "Unable to acquire lock" or "Port 3000 is in use":**  
Run this once to kill the old dev server and remove the lock, then start fresh:

```bash
npm run dev:fresh
```

Or manually: `npm run dev:kill` then `npm run dev`.  
On Windows you can also run `.\scripts\dev-fresh.ps1` in PowerShell.

### Demo mode (judge-friendly)

On the login page, click **“Try demo (instant dashboard)”**.

This will:

- seed a demo user into MongoDB
- populate budgets, goals, and realistic transactions
- log you in automatically

### Key routes

- `/` — Landing page
- `/login`, `/signup`, `/forgot` — Auth
- `/dashboard` — Smart dashboard with VisualPe-style layout:
  - **Assets & Liabilities** card (cash, investments, credit, net worth)
  - **Portfolio Breakdown** (equity, mutual funds, crypto, bonds) with interactive pie chart
  - **UPI & Payments** view (received/sent with recent transactions)
  - **KPIs** (health score, balance, income, expenses)
  - **Expense breakdown** pie chart
  - **Income vs Expenses** bar chart
  - **Investment growth** line chart
  - **Risk meter** gauge
  - **Smart insights** panel
  - **Notifications** panel
  - **Android simulator** preview
- `/goals` — Goal tracker
- `/settings` — Security + integrations demo settings

### API endpoints (high level)

- `POST /api/auth/signup`, `POST /api/auth/login`, `POST /api/auth/logout`, `GET /api/auth/me`
- `GET/POST /api/transactions`, `PATCH/DELETE /api/transactions/:id`
- `GET/PUT /api/budgets`
- `GET/POST /api/goals`, `PATCH/DELETE /api/goals/:id`
- `GET /api/dashboard/summary`
- `GET /api/insights`
- `GET /api/notifications`
- `GET /api/report/pdf`
- `POST /api/demo/seed`, `POST /api/demo/login`

### Docker (full stack)

Build and run app + MongoDB:

```bash
docker compose up --build
```

Then open `http://localhost:3000`.

### Notes

- Auth is **JWT in an HttpOnly cookie** (middleware guards protected routes).
- AI insights are **deterministic, rule-based** with a placeholder for optional LLM augmentation.
- This repo ships **local-first**: no external paid APIs required for the core demo.

