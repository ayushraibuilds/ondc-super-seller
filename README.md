# 🛒 ONDC Super Seller

> Let Indian shopkeepers manage their ONDC catalog through WhatsApp — in Hindi, English, or Hinglish.

[![CI](https://github.com/ondc-super-seller/actions/workflows/ci.yml/badge.svg)](https://github.com/ondc-super-seller/actions/workflows/ci.yml)

A WhatsApp-native inventory management system that converts natural language messages (voice notes, images, and text) into [Beckn protocol](https://beckn.network/) compliant catalogs on the [ONDC](https://ondc.org/) network. A real-time Next.js dashboard provides full visibility, analytics, and CRUD control.

---

## ✨ Features

### WhatsApp Integration
- 🗣️ **Voice note support** — speak in Hindi/Hinglish, AI transcribes and processes
- 📸 **Image recognition** — send product photos, AI extracts details
- 💬 **Natural language CRUD** — add, update, delete products via text
- 🔤 **Multilingual** — Hindi, English, Hinglish supported natively
- ⚠️ **Low stock alerts** — automatic WhatsApp notifications when inventory runs low

### Dashboard
- 📊 **Real-time inventory** — SSE live updates, search, pagination
- 💰 **Price intelligence** — market price comparison with competitive analysis
- 📈 **Analytics** — revenue trends, category breakdown, top products
- 🛒 **Order management** — view, filter, and manage ONDC orders
- 📋 **Activity logs** — full audit trail of all WhatsApp and dashboard actions
- 🔔 **Notification center** — in-app alerts for price changes, low stock, orders
- 🌐 **Multi-language UI** — English ↔ Hindi toggle
- 🌙 **Theme** — dark/light/system auto-detect
- 📱 **PWA** — installable on mobile with app shortcuts
- 📦 **CSV import/export** — bulk catalog management
- ⚡ **Batch price adjustment** — match market prices in one click

### Infrastructure
- 🔐 **JWT authentication** — Supabase-powered auth with seller profiles
- 🧪 **130 automated tests** — unit + integration tests
- 🐳 **Docker Compose** — single-command local dev setup
- 🔄 **CI/CD** — GitHub Actions for pytest + Next.js build
- 🌐 **ONDC sandbox** — protocol-compliant `/search`, `/select`, `/confirm` endpoints

---

## 🏗 Architecture

```
📱 WhatsApp (Voice / Image / Text)
        │
        ▼
🔀 FastAPI Webhook (/whatsapp-webhook)
   └── Twilio Signature Validation
        │
        ▼
🧠 LangGraph Agent (Groq LLM)
   ├── Intent Classifier → ADD / UPDATE / DELETE / UNKNOWN
   ├── Entity Extractor (Pydantic structured output)
   ├── Smart Upsert (fuzzy name matching > 0.7 similarity)
   └── Input Sanitizer (HTML strip, price validation)
        │
        ▼
💾 Supabase (PostgreSQL + Auth)
        │
        ▼
📊 Next.js Dashboard (SSE live updates)
   ├── Inventory table with search & CSV export
   ├── Price intelligence with market comparison
   ├── Analytics (revenue, categories, trends)
   ├── Order management
   ├── ONDC sandbox status
   └── Notification center + Activity logs
```

---

## 🛠 Tech Stack

| Layer | Technology |
|---|---|
| **AI Agent** | LangGraph + Groq (Llama 3) + Pydantic |
| **Backend** | FastAPI + Twilio SDK + SlowAPI rate limiter |
| **Database** | Supabase (PostgreSQL + Auth + Storage) |
| **Dashboard** | Next.js 16 + Tailwind CSS v4 + Framer Motion |
| **Testing** | pytest (130 tests) |
| **DevOps** | Docker Compose + GitHub Actions CI |
| **Protocols** | ONDC/Beckn compliant catalog schema |

---

## 🚀 Quick Start

### Prerequisites
- Python 3.12+
- Node.js 20+
- Supabase project (free tier works)
- Groq API key (free tier works)
- Twilio account (for WhatsApp)

### Option 1: Docker Compose (Recommended)

```bash
# Clone the repo
git clone https://github.com/ondc-super-seller.git
cd ondc-super-seller

# Configure environment
cp backend/.env.example backend/.env
# Edit backend/.env with your Supabase, Groq, and Twilio credentials

# Start everything
docker compose up
```

Backend → `http://localhost:8000` | Dashboard → `http://localhost:3000`

### Option 2: Manual Setup

#### Backend

```bash
cd backend

# Create virtual environment
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Configure environment
cp .env.example .env
# Edit .env with your credentials

# Start server
uvicorn server:app --host 0.0.0.0 --port 8000 --reload
```

#### Dashboard

```bash
cd dashboard

# Install dependencies
npm install

# Start dev server
npm run dev
```

#### WhatsApp Webhook

```bash
# Start ngrok tunnel
ngrok http 8000

# Set Twilio webhook URL to:
# https://your-ngrok-url.ngrok.io/whatsapp-webhook
```

---

## 🔌 API Reference

### Core Endpoints

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| `POST` | `/whatsapp-webhook` | Twilio Sig | Receive WhatsApp messages (text, voice, image) |
| `GET` | `/api/v1/catalog` | JWT | Fetch catalog (supports `limit`, `offset`, `seller_id`) |
| `GET` | `/api/v1/catalog/stream` | JWT | SSE live catalog stream |
| `POST` | `/api/v1/catalog/item` | JWT | Add a product |
| `PUT` | `/api/v1/catalog/item/{id}` | JWT | Update a product |
| `DELETE` | `/api/v1/catalog/item/{id}` | JWT | Delete a product |
| `POST` | `/api/v1/catalog/import/csv` | JWT | Bulk import from CSV |

### Price Intelligence

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| `GET` | `/api/v1/catalog/price-check` | JWT | Market price comparison report |
| `GET` | `/api/catalog/export/csv` | JWT | Export catalog as CSV |
| `POST` | `/api/catalog/batch-price-update` | JWT | Batch update prices to match market |

### Orders & Sellers

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| `GET` | `/api/v1/orders` | JWT | List orders |
| `POST` | `/api/v1/orders` | JWT | Create order |
| `GET` | `/api/v1/sellers` | JWT | List sellers |
| `GET/PUT` | `/api/v1/seller/profile` | JWT | Seller profile CRUD |

### ONDC Protocol

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| `POST` | `/api/ondc/search` | None | Beckn `/search` handler |
| `POST` | `/api/ondc/select` | None | Beckn `/select` handler |
| `POST` | `/api/ondc/confirm` | None | Beckn `/confirm` handler |
| `GET` | `/api/v1/ondc/status` | None | ONDC sandbox connection status |

### Other

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| `POST` | `/api/catalog/item/{id}/image` | JWT | Upload product image |
| `GET` | `/api/activity` | JWT | Activity/webhook logs |
| `GET` | `/health` | None | Health check |
| `POST` | `/api/signup` | None | Create seller account |
| `POST` | `/api/login` | None | Authenticate seller |

---

## 🧪 Testing

```bash
cd backend
source venv/bin/activate
pytest tests/ -v
```

**130 tests** covering:
- ONDC adapter (Beckn protocol compliance)
- Webhook integration (text, voice, image messages)
- Catalog CRUD operations
- Order management
- Price reference engine
- Authentication & rate limiting
- Seller profile management

---

## 📁 Project Structure

```
ondc-super-seller/
├── backend/
│   ├── server.py              # FastAPI app + CORS + rate limiting
│   ├── db.py                  # Supabase database operations
│   ├── langgraph_agent.py     # AI agent (intent + entity extraction)
│   ├── price_reference.py     # Market price comparison engine
│   ├── ondc_adapter.py        # Beckn protocol adapter
│   ├── routes/
│   │   ├── webhook.py         # WhatsApp webhook (Twilio)
│   │   ├── catalog.py         # Catalog CRUD + CSV + price check
│   │   ├── orders.py          # Order management
│   │   ├── sellers.py         # Seller profiles
│   │   ├── ondc.py            # ONDC /search /select /confirm
│   │   ├── images.py          # Product image upload
│   │   └── auth.py            # JWT auth middleware
│   ├── tests/                 # 12 test files, 130 tests
│   ├── requirements.txt
│   └── Dockerfile
├── dashboard/
│   ├── src/
│   │   ├── app/
│   │   │   ├── dashboard/     # Main inventory dashboard
│   │   │   ├── analytics/     # Revenue & category analytics
│   │   │   ├── price-check/   # Price intelligence + market comparison
│   │   │   ├── orders/        # Order management
│   │   │   ├── import/        # CSV import
│   │   │   ├── logs/          # Activity & webhook logs
│   │   │   ├── seller/        # Seller profile
│   │   │   ├── onboarding/    # Multi-step seller onboarding
│   │   │   ├── login/         # Authentication
│   │   │   └── signup/        # Registration
│   │   └── components/
│   │       ├── InventoryTable  # Live inventory with search
│   │       ├── NotificationCenter  # In-app notifications
│   │       ├── ActivityLog     # Real-time activity feed
│   │       ├── ThemeToggle     # Dark/light/system theme
│   │       ├── LangToggle      # EN/Hindi language switch
│   │       ├── Sparkline       # SVG trend charts
│   │       └── ...            # StatCards, ProductModal, etc.
│   ├── public/
│   │   ├── manifest.json      # PWA manifest with shortcuts
│   │   └── sw.js              # Service worker (static cache only)
│   ├── package.json
│   └── Dockerfile
├── docker-compose.yml         # Single-command dev setup
├── .github/workflows/ci.yml   # GitHub Actions CI pipeline
└── .gitignore
```

---

## 💡 How It Works

1. **Shopkeeper sends a WhatsApp message** like:  
   *"Bhaiya, 10 kilo Aashirvaad atta ka price 450 rupees rakh do"*

2. **AI Agent** processes the message:
   - Intent Classifier → `ADD`
   - Entity Extractor → `name: "Aashirvaad Atta", price: 450, qty: 10, unit: "kg"`

3. **Smart Upsert** checks for existing items via fuzzy matching (>70% similarity)

4. **Beckn Catalog** is generated with ONDC-compliant schema

5. **Dashboard** receives the update via SSE and displays it instantly

6. **Low stock alerts** are sent automatically when inventory drops below threshold

---

## 🤝 Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for development guidelines.

---

## 📄 License

MIT
