# Mikolo Simplified — API (Contribution Management)

Uganda-first ceremony operating system. This is the backend for the first module: **Contribution Management** — pledges, payments, receipts, thank-yous, reconciliation, duplicate detection, reports, and a full audit trail. Built to extend cleanly into Committees, Tasks, Budgets, Expenses, Vendors, and the Ceremony Programme.

## Stack

- Node.js + Express + TypeScript
- PostgreSQL + Prisma ORM
- JWT authentication, bcrypt password hashing
- CSV import/export, printable HTML receipts

## Local setup

```bash
cp .env.example .env
# edit .env — point DATABASE_URL at a local or hosted Postgres instance

npm install
npm run prisma:migrate:dev   # creates tables
npm run seed                 # optional: demo event + contributors
npm run dev                  # http://localhost:4000
```

Seed login: `admin@mikolo.ug` / `Password123!`

## Deploying to Render

1. Push this folder to a GitHub repo.
2. In Render, choose **New > Blueprint** and point it at the repo — `render.yaml` provisions a free Postgres database and a web service automatically.
3. Or manually: create a Postgres instance, then a Web Service with:
   - Build command: `npm install && npm run build`
   - Start command: `npm run prisma:migrate && npm start`
   - Env vars: `DATABASE_URL`, `JWT_SECRET`, `CORS_ORIGIN` (your Vercel frontend URL), `NODE_ENV=production`
4. Once live, hit `https://<your-service>.onrender.com/health` to confirm it's up.

## API overview

All routes except `/health`, `/api/auth/register`, and `/api/auth/login` require `Authorization: Bearer <token>`.

| Area | Routes |
|---|---|
| Auth | `POST /api/auth/register`, `POST /api/auth/login`, `GET /api/auth/me` |
| Events | `POST/GET/PATCH /api/events` |
| Categories | `GET/POST/DELETE /api/categories` |
| Contributors | `GET/POST/PATCH/DELETE /api/contributors`, `GET /api/contributors/export/csv`, `POST /api/contributors/import/csv` |
| Pledges | `POST/PATCH/DELETE /api/pledges` |
| Payments | `POST/PATCH/DELETE /api/payments`, `POST /api/payments/:id/reconcile`, `POST /api/payments/:id/reverse`, `GET /api/payments/:id/receipt` |
| Reports | `GET /api/reports/summary`, `GET /api/reports/timeline` |
| Reconciliation | `GET /api/reconciliation/pending`, `GET /api/reconciliation/duplicates`, `POST /api/reconciliation/duplicates/:id/merge` |
| Audit | `GET /api/audit-logs` |
| Thank-you | `POST /api/thank-you`, `POST /api/thank-you/bulk`, `GET /api/thank-you` |

Full request/response shapes are visible directly in `src/routes/*.ts` — each file is short and self-contained.

## Notes for going to production

- Swap the printable-HTML receipt for a proper PDF (e.g. `pdf` skill / `puppeteer`) once you're ready.
- Thank-you messages are queued but not actually sent — wire up an SMS/WhatsApp provider (Africa's Talking, Twilio) in `thankyou.routes.ts`.
- Add rate limiting and request logging aggregation before opening this up publicly.
