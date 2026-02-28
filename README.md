# 🛒 ONDC Super Seller

> Let Indian shopkeepers manage their ONDC catalog through WhatsApp — in Hinglish.

A WhatsApp-native inventory management system that converts natural language messages (Hindi, English, Hinglish) into [Beckn protocol](https://beckn.network/) compliant catalogs on the [ONDC](https://ondc.org/) network. A real-time Next.js dashboard provides full visibility and CRUD control.

## Architecture

```
📱 WhatsApp Message (Hinglish/English)
        │
        ▼
🔀 FastAPI Webhook (/whatsapp-webhook)
   └── Twilio Signature Validation
        │
        ▼
🧠 LangGraph Agent (Ollama llama3.1)
   ├── Intent Classifier → ADD / UPDATE / DELETE / UNKNOWN
   ├── Product Entity Extractor (Pydantic structured output)
   ├── Smart Upsert (fuzzy name matching > 0.7 similarity)
   └── Input Sanitizer (HTML strip, price validation)
        │
        ▼
💾 SQLite (thread-safe with locking)
        │
        ▼
📊 Next.js Dashboard (SSE live updates)
   ├── Real-time inventory table with search & pagination
   ├── Multi-seller selector
   ├── Stat cards (total products, value, low stock alerts)
   └── Add / Edit / Delete via REST API (API key protected)
```

## Tech Stack

| Layer | Technology |
|---|---|
| AI Agent | LangGraph + Ollama (llama3.1) + Pydantic |
| API Server | FastAPI + Twilio SDK |
| Database | SQLite (thread-safe) |
| Dashboard | Next.js 16 + Tailwind CSS v4 + Framer Motion |
| Tunnel | ngrok (for WhatsApp webhook) |

## Setup

### Prerequisites
- Python 3.10+
- Node.js 18+
- [Ollama](https://ollama.ai/) with `llama3.1` model pulled
- ngrok (for WhatsApp webhook)

### Backend

```bash
cd backend

# Create and activate virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Configure environment
cp .env.example .env  # or edit .env directly
# Set: LLM_MODEL, API_KEY, TWILIO_AUTH_TOKEN (optional)

# Start Ollama (in a separate terminal)
ollama run llama3.1

# Start the server
python start_server.py
```

The API will be available at `http://localhost:8000`.

### Dashboard

```bash
cd dashboard

# Install dependencies
npm install

# Start dev server
npm run dev
```

The dashboard will be at `http://localhost:3000`.

### WhatsApp Integration

```bash
# Start ngrok tunnel
ngrok http 8000

# Copy the ngrok URL and set it as your Twilio webhook:
# https://your-ngrok-url.ngrok.io/whatsapp-webhook
```

## API Endpoints

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| POST | `/whatsapp-webhook` | Twilio Signature | Receive WhatsApp messages |
| GET | `/api/catalog` | None | Fetch catalog (supports `limit`, `offset`, `seller_id`) |
| GET | `/api/catalog/stream` | None | SSE live catalog stream |
| GET | `/api/sellers` | None | List all seller IDs |
| POST | `/api/catalog/item` | API Key | Add a product |
| PUT | `/api/catalog/item/{id}` | API Key | Update a product |
| DELETE | `/api/catalog/item/{id}` | API Key | Delete a product |
| GET | `/health` | None | Health check |

## Testing

```bash
cd backend
source venv/bin/activate
pytest test_hinglish.py test_intent.py test_update.py -v
```

> **Note:** Tests require Ollama running locally with `llama3.1`. They are integration tests that invoke the actual LLM.

## How It Works

1. **Shopkeeper sends a WhatsApp message** like: *"Bhaiya, 10 kilo Aashirvaad atta ka price 450 rupees rakh do"*
2. **Intent Classifier** detects this is an ADD operation
3. **Entity Extractor** parses: name="Aashirvaad Atta", price=450, quantity=10, unit="kg"
4. **Smart Upsert** checks for existing items with similar names (fuzzy matching)
5. **Beckn Catalog** is generated/updated with proper ONDC schema
6. **Dashboard** receives the update via SSE and displays it in real-time

## License

MIT
