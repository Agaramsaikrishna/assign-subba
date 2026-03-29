# Tool Subscription Management Dashboard

A full-stack backend system to **track, manage, and optimize** your organization's software tool subscriptions — with an AI-powered assistant, real-time analytics, and automated renewal reminders.

---

## Table of Contents

- [Features](#features)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Quick Start](#quick-start)
- [Environment Variables](#environment-variables)
- [API Reference](#api-reference)
- [Frontend](#frontend)
- [AI Assistant](#ai-assistant)
- [Automated Reminders](#automated-reminders)
- [Database Schema](#database-schema)

---

## Features

| Feature | Details |
|---|---|
| **Auth** | JWT Bearer tokens — register, login, protected routes |
| **Subscription CRUD** | Add, edit, delete, search, filter, paginate subscriptions |
| **Dashboard** | 12+ metrics: annual cost, overdue alerts, category breakdown, top-5 expensive |
| **Analytics** | Spend trends, cost optimization tips, renewal density calendar |
| **AI Assistant** | Natural language queries via LangChain + Groq (llama-3.3-70b) with 7 live tools |
| **Web Search** | Real-time tool pricing and alternatives via DuckDuckGo |
| **Memory** | Short-term (in-memory last 10 messages) + long-term (SQLite UserPreference) |
| **Reminders** | APScheduler — automated 7-day, 30-day, and overdue renewal notifications |
| **Frontend** | Single-page app: Dashboard, Subscriptions, Analytics, AI Chat — no framework needed |

---

## Tech Stack

| Layer | Technology |
|---|---|
| **API Framework** | FastAPI 0.100+ |
| **Database** | SQLite via SQLAlchemy ORM |
| **Auth** | JWT (python-jose) + bcrypt password hashing |
| **AI Framework** | LangChain 1.x + LangGraph ReAct agent |
| **LLM Provider** | Groq (`llama-3.3-70b-versatile`) — or OpenAI (`gpt-4o`) via env toggle |
| **Web Search** | DuckDuckGo Search (langchain-community) |
| **Scheduler** | APScheduler BackgroundScheduler |
| **Frontend** | Vanilla JS + Tailwind CSS CDN + Chart.js + Font Awesome |
| **Server** | Uvicorn (ASGI) |

---

## Project Structure

```
assign-subbu-anna/
├── app.py                        # FastAPI entry point — routers, CORS, lifespan, frontend serving
├── database.py                   # SQLAlchemy engine, SessionLocal, Base, get_db dependency
├── models.py                     # ORM models: User, Subscription, ChatMessage, UserPreference
├── schemas.py                    # Pydantic schemas: request/response validation
├── auth.py                       # JWT utilities: create_access_token, get_current_user
├── requirements.txt              # All Python dependencies
├── .env                          # Environment variables (API keys, secret key)
│
├── routers/
│   ├── __init__.py
│   ├── auth.py                   # POST /auth/register, POST /auth/login, GET /auth/me
│   ├── subscriptions.py          # Full CRUD + filtering + pagination for /subscriptions
│   ├── dashboard.py              # GET /dashboard — 12+ aggregated metrics
│   ├── analytics.py              # GET /analytics, GET /analytics/summary
│   └── chat.py                   # POST /chat, GET /chat/history, DELETE /chat/history
│
├── services/
│   ├── __init__.py
│   ├── chat_service.py           # LangChain ReAct agent with 7 tools + LLM factory
│   ├── memory_service.py         # Short-term + long-term conversation memory
│   ├── scheduler.py              # APScheduler jobs: 7-day, 30-day, overdue alerts
│   └── search_service.py         # DuckDuckGo web search wrapper
│
├── frontend/
│   └── index.html                # Single-page app (SPA) — served at GET /app
│
├── subscriptions.db              # Auto-created SQLite database
└── smoke_test.py                 # Quick import/sanity check script
```

---

## Quick Start

### 1. Prerequisites

- Python 3.10+
- A free [Groq API key](https://console.groq.com) (or OpenAI API key)

### 2. Clone and set up

```bash
git clone <your-repo-url>
cd assign-subbu-anna
```

### 3. Create virtual environment

```bash
python -m venv .venv

# Windows
.venv\Scripts\activate

# macOS / Linux
source .venv/bin/activate
```

### 4. Install dependencies

```bash
pip install -r requirements.txt
```

### 5. Configure environment

Create a `.env` file in the project root (or edit the existing one):

```env
GROQ_API_KEY=gsk_your_groq_key_here
SECRET_KEY=your-secret-jwt-key-change-this-in-production

# LLM Configuration
MODEL_PROVIDER=groq                    # groq (default) | openai
LLM_MODEL=llama-3.3-70b-versatile     # or gpt-4o for OpenAI
# OPENAI_API_KEY=sk-...               # only needed if MODEL_PROVIDER=openai
```

### 6. Run the server

```bash
uvicorn app:app --reload
```

Server starts at: `http://localhost:8000`

### 7. Access the app

| URL | Description |
|---|---|
| `http://localhost:8000/app` | Frontend dashboard (SPA) |
| `http://localhost:8000/docs` | Swagger UI — interactive API explorer |
| `http://localhost:8000/redoc` | ReDoc API documentation |
| `http://localhost:8000/` | API info JSON |

---

## Environment Variables

| Variable | Required | Default | Description |
|---|---|---|---|
| `GROQ_API_KEY` | Yes (if Groq) | — | Groq API key from console.groq.com |
| `SECRET_KEY` | Yes | — | JWT signing secret — change in production |
| `MODEL_PROVIDER` | No | `groq` | `groq` or `openai` |
| `LLM_MODEL` | No | `llama-3.3-70b-versatile` | Model name for chosen provider |
| `OPENAI_API_KEY` | Only if OpenAI | — | OpenAI API key |

---

## API Reference

All endpoints except `/auth/*`, `/chat/prompts`, and `/` require a **Bearer token** in the `Authorization` header.

### Authentication

| Method | Path | Auth | Description |
|---|---|---|---|
| `POST` | `/auth/register` | No | Register with email + password, returns JWT token |
| `POST` | `/auth/login` | No | Login, returns JWT token |
| `GET` | `/auth/me` | Yes | Returns current user info |

**Register example:**
```bash
curl -X POST http://localhost:8000/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email": "user@example.com", "password": "secret123"}'
```

**Response:**
```json
{
  "access_token": "eyJhbGc...",
  "token_type": "bearer"
}
```

---

### Subscriptions

| Method | Path | Auth | Description |
|---|---|---|---|
| `GET` | `/subscriptions/` | Yes | List with filters, sorting, pagination |
| `GET` | `/subscriptions/{id}` | Yes | Get single subscription |
| `POST` | `/subscriptions/` | Yes | Create new subscription |
| `PUT` | `/subscriptions/{id}` | Yes | Update subscription |
| `DELETE` | `/subscriptions/{id}` | Yes | Delete subscription |

**Query Parameters for `GET /subscriptions/`:**

| Param | Type | Description |
|---|---|---|
| `status` | string | Filter: `active` \| `inactive` \| `cancelled` |
| `category` | string | Filter by category |
| `billing_cycle` | string | Filter: `monthly` \| `yearly` |
| `search` | string | Search tool name (case-insensitive) |
| `renewing_in_days` | int | Only subscriptions renewing within N days |
| `sort_by` | string | `cost` \| `renewal_date` \| `tool_name` \| `created_at` |
| `sort_order` | string | `asc` \| `desc` |
| `page` | int | Page number (default: 1) |
| `page_size` | int | Items per page (default: 20, max: 100) |

**Subscription fields:**

| Field | Type | Required | Values |
|---|---|---|---|
| `tool_name` | string | Yes | e.g., "GitHub", "Slack" |
| `purchase_date` | date | Yes | `YYYY-MM-DD` |
| `billing_cycle` | string | Yes | `monthly` \| `yearly` |
| `renewal_date` | date | Yes | `YYYY-MM-DD` |
| `cost` | float | Yes | e.g., `49.99` |
| `category` | string | No | `DevOps` \| `Communication` \| `Productivity` \| `Security` \| `Analytics` \| `Design` \| `Other` |
| `status` | string | No | `active` \| `inactive` \| `cancelled` |
| `description` | string | No | Free text |
| `currency` | string | No | Default: `USD` |

---

### Dashboard

| Method | Path | Auth | Description |
|---|---|---|---|
| `GET` | `/dashboard/` | Yes | Full analytics — 12+ metrics |

**Response includes:**
- `total_subscriptions`, `active`, `inactive`, `cancelled` counts
- `total_monthly_cost`, `total_yearly_cost`, `total_annual_cost`, `daily_cost`
- `upcoming_7_days` — list of tools renewing this week
- `upcoming_30_days_count` — count renewing this month
- `overdue_count`, `overdue_tools` — past renewal date
- `top_5_expensive` — highest cost active subscriptions
- `spend_by_category` — cost breakdown per category

---

### Analytics

| Method | Path | Auth | Description |
|---|---|---|---|
| `GET` | `/analytics/` | Yes | Deep analytics: trends, optimization tips, renewal density |
| `GET` | `/analytics/summary` | Yes | Quick plain-number spend summary |

**`/analytics/` response includes:**
- `spend_trends` — monthly spend over time
- `potential_savings` — cost optimization recommendations
- `renewal_density` — how many renewals per month
- `category_analysis` — per-category stats
- `billing_cycle_distribution` — monthly vs yearly split

---

### Chat (AI Assistant)

| Method | Path | Auth | Description |
|---|---|---|---|
| `POST` | `/chat/` | Yes | Chat with AI assistant |
| `GET` | `/chat/prompts` | No | 8 predefined quick-access prompts |
| `GET` | `/chat/history` | Yes | Conversation history |
| `DELETE` | `/chat/history` | Yes | Clear history and reset memory |

**Chat request:**
```json
{
  "message": "What are my most expensive subscriptions this month?"
}
```

**Chat response:**
```json
{
  "reply": "Your top 3 most expensive subscriptions are: ..."
}
```

---

## Frontend

The frontend is a single-page application served directly by FastAPI.

**URL:** `http://localhost:8000/app`

### Views

| View | Description |
|---|---|
| **Dashboard** | KPI cards (annual cost, active subs, overdue, renewing this week), category donut chart (Chart.js), top-5 expensive table, 7-day renewal list |
| **Subscriptions** | Filter bar (search/status/category/billing), sortable table, add/edit modal, delete confirm — full CRUD |
| **Analytics** | Monthly spend trend (line chart), renewal density (bar chart), optimization tips, spend summary cards |
| **AI Chat** | Quick prompt chips, live chat interface with typing indicator, connected to LangChain agent backend |

### Auth Flow

1. Open `http://localhost:8000/app`
2. Register or login — JWT token saved to `localStorage`
3. All API calls automatically include `Authorization: Bearer <token>`
4. Logout clears the token and returns to auth screen

---

## AI Assistant

The AI assistant uses a **LangGraph ReAct agent** (reasoning + acting loop) with 7 live tools:

| Tool | Description |
|---|---|
| `get_all_subscriptions` | Fetch all subscriptions with optional status filter |
| `get_dashboard_summary` | Full analytics: annual cost, overdue, category breakdown |
| `get_upcoming_renewals` | Subscriptions renewing within N days |
| `get_expensive_tools` | Top N most expensive active subscriptions |
| `get_overdue_subscriptions` | Subscriptions past their renewal date |
| `get_spend_by_category` | Cost breakdown per category |
| `search_web_for_tool_info` | DuckDuckGo search for real-time pricing and alternatives |

**Example questions you can ask:**
- "What is my total annual spend?"
- "Which subscriptions are renewing this week?"
- "Show me all overdue subscriptions"
- "What are cheaper alternatives to Salesforce?"
- "Which category am I spending the most on?"
- "Suggest ways to reduce my software costs"

**LLM Providers:**

| Provider | Model | How to enable |
|---|---|---|
| Groq (default) | `llama-3.3-70b-versatile` | Set `GROQ_API_KEY` in `.env` |
| OpenAI | `gpt-4o` | Set `MODEL_PROVIDER=openai` and `OPENAI_API_KEY` in `.env` |

---

## Automated Reminders

APScheduler runs background jobs on startup and every 24 hours:

| Job | Schedule | Trigger |
|---|---|---|
| 7-day renewal check | Every 24 hours | Subscription renewing within 7 days |
| 30-day renewal check | Every 24 hours | Subscription renewing in 8–30 days |
| Overdue check | Every 12 hours | Subscription past renewal date |

Notifications are printed to the server console (mock email + webhook):

```
============================================================
[EMAIL NOTIFICATION]
  To      : user@example.com
  Subject : Subscription Renewal in 3 day(s): GitHub
  Body    : Your subscription to GitHub (DevOps) renews on 2025-04-01 ...
============================================================
```

---

## Database Schema

SQLite database (`subscriptions.db`) is auto-created on first run.

```
users
  id, email (unique), hashed_password, created_at

subscriptions
  id, user_id (FK), tool_name, purchase_date, billing_cycle,
  renewal_date, cost, category, status, description, currency, created_at

chat_messages
  id, user_id (FK), role (user|assistant), content, created_at

user_preferences
  id, user_id (FK), key, value
```

---

## Development Notes

- **CORS** is set to `allow_origins=["*"]` — restrict in production
- **JWT secret** in `.env` must be changed for production
- **SQLite** is used for simplicity — swap for PostgreSQL for production
- The database file `subscriptions.db` is auto-created; delete it to reset all data
- `smoke_test.py` can be run to verify imports and basic sanity

---

## License

MIT
