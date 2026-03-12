# ONDC Super Seller

[![CI](https://github.com/ayushraibuilds/ondc-super-seller/actions/workflows/ci.yml/badge.svg)](https://github.com/ayushraibuilds/ondc-super-seller/actions/workflows/ci.yml)

WhatsApp-first inventory management for sellers. Sellers can add, update, and review catalog inventory through text messages, voice notes, and product images, while the dashboard gives them visibility into stock, pricing, activity, imports, and alerts.

## What the product already does

- WhatsApp catalog updates through natural language text
- Voice-note parsing for inventory updates
- Image-based product extraction and catalog updates
- Seller dashboard with inventory visibility and CRUD controls
- CSV import and export for bulk catalog maintenance
- Price intelligence and batch price updates
- Low-stock alerts and activity history
- Seller profile management and order tracking
- Production hardening for auth, Twilio validation, health checks, and deployment

## Product positioning

This product is for sellers maintaining inventory, not a seller-acquisition or storefront tool. The core workflow is:

1. A seller sends a WhatsApp message, voice note, or image.
2. The backend parses the request and updates the seller catalog.
3. The dashboard reflects the inventory state, pricing status, recent activity, and low-stock issues.

## Stack

- Backend: FastAPI, Supabase, Twilio, Redis, Celery
- AI workflow: Groq-backed parsing with retries, timeout handling, and a circuit breaker
- Dashboard: Next.js, Tailwind CSS, Framer Motion
- Infrastructure: Railway for backend services, Vercel for dashboard, Docker Compose for local/prod-style self-hosting

## Local development

### Prerequisites

- Python 3.12+
- Node.js 20+
- A Supabase project
- A Groq API key
- A Twilio WhatsApp sender or sandbox

### Backend

```bash
cd backend
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
cp .env.example .env
uvicorn server:app --host 0.0.0.0 --port 8000 --reload
```

### Dashboard

```bash
cd dashboard
npm install
npm run dev
```

### Docker

```bash
cp backend/.env.example backend/.env
docker compose up
```

Backend: `http://localhost:8000`  
Dashboard: `http://localhost:3000`

## Production deployment

### Current deployment model

- Backend API: Railway
- Celery worker: Railway
- Celery beat: Railway
- Dashboard: Vercel

### Deployment config in this repo

- [backend/railway.json](/Users/dankmagician/Documents/New%20project/ondc-super-seller/backend/railway.json)
- [dashboard/vercel.json](/Users/dankmagician/Documents/New%20project/ondc-super-seller/dashboard/vercel.json)
- [.env.production.example](/Users/dankmagician/Documents/New%20project/ondc-super-seller/.env.production.example)
- [DEPLOY.md](/Users/dankmagician/Documents/New%20project/ondc-super-seller/DEPLOY.md)
- [PROD_READY_CHECKLIST.md](/Users/dankmagician/Documents/New%20project/ondc-super-seller/PROD_READY_CHECKLIST.md)

### Railway backend start commands

Keep these in the Railway UI because the API, worker, and beat all share the same `backend/` root:

- API: `uvicorn server:app --host 0.0.0.0 --port 8000 --workers 4`
- Worker: `celery -A celery_app worker --loglevel=info --concurrency=2`
- Beat: `celery -A celery_app beat --loglevel=info`

### Required production variables

Backend / Railway:

- `NODE_ENV=production`
- `PUBLIC_URL=https://your-backend.up.railway.app`
- `SUPABASE_URL`
- `SUPABASE_ANON_KEY`
- `SUPABASE_SERVICE_ROLE_KEY`
- `GROQ_API_KEY`
- `JWT_SECRET`
- `TWILIO_ACCOUNT_SID`
- `TWILIO_AUTH_TOKEN`
- `TWILIO_WHATSAPP_FROM`
- `REDIS_URL`
- `CORS_ORIGINS=https://your-dashboard.vercel.app`

Dashboard / Vercel:

- `NEXT_PUBLIC_API_URL=https://your-backend.up.railway.app`

Twilio inbound webhook:

- `https://your-backend.up.railway.app/whatsapp-webhook`

The current Railway backend public URL used during deployment validation is:

- [https://ondc-super-seller-production.up.railway.app](https://ondc-super-seller-production.up.railway.app)

## Testing

Backend test suite:

```bash
cd backend
source venv/bin/activate
pytest tests/ -q
```

Current backend regression status during the production-hardening pass: `140 passed`.

Dashboard checks:

```bash
cd dashboard
npm run build
npm run lint
```

## Remaining production gaps

The main remaining items are tracked in [PROD_READY_CHECKLIST.md](/Users/dankmagician/Documents/New%20project/ondc-super-seller/PROD_READY_CHECKLIST.md). The biggest open items are:

- rotate previously exposed secrets and revoke old values
- add stronger observability and alerting
- finish frontend accessibility cleanup
- decide whether ONDC stays sandbox-only or becomes a real production integration

## Legal

- [Privacy Policy](/Users/dankmagician/Documents/New%20project/ondc-super-seller/dashboard/src/app/privacy/page.tsx)
- [Terms of Service](/Users/dankmagician/Documents/New%20project/ondc-super-seller/dashboard/src/app/terms/page.tsx)

## License

MIT
