# 📋 ChatApp - Complete Project Implementation Plan

## 🎯 Project Overview

A full-stack **ChatGPT-like application** built for a data science course (298B), featuring intelligent model routing, real-time streaming, and production-ready architecture.

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                           FRONTEND                                   │
│                   React + Vite + TypeScript                         │
│                   TailwindCSS + shadcn/ui                           │
│                      Port: 5173/5174                                │
└─────────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────────┐
│                           BACKEND                                    │
│                    FastAPI (Python 3.11)                            │
│                        Port: 8000                                   │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────────────┐    │
│  │   Auth   │  │  Router  │  │  Safety  │  │    LLM Client    │    │
│  │(Firebase)│  │  (MoE)   │  │  Layer   │  │   (OpenRouter)   │    │
│  └──────────┘  └──────────┘  └──────────┘  └──────────────────┘    │
└─────────────────────────────────────────────────────────────────────┘
                    │                              │
          ┌────────┴────────┐                     │
          ▼                 ▼                     ▼
    ┌──────────┐      ┌──────────┐      ┌─────────────────┐
    │PostgreSQL│      │  Redis   │      │   OpenRouter    │
    │Port: 5433│      │Port: 6379│      │   (LLM API)     │
    └──────────┘      └──────────┘      └─────────────────┘
```

---

## 📁 Project Structure

```
298B/
├── app/
│   ├── backend/                    # FastAPI Backend
│   │   ├── app/
│   │   │   ├── api/v1/            # API Endpoints
│   │   │   │   ├── auth.py        # Authentication
│   │   │   │   ├── chat.py        # Chat endpoints (send, stream)
│   │   │   │   ├── conversations.py # CRUD operations
│   │   │   │   ├── health.py      # Health checks
│   │   │   │   ├── admin.py       # Admin endpoints
│   │   │   │   └── deps.py        # Dependencies
│   │   │   ├── core/              # Core utilities
│   │   │   │   ├── config.py      # Settings/configuration
│   │   │   │   ├── models_hidden.py # Hidden model IDs
│   │   │   │   ├── security.py    # Firebase auth
│   │   │   │   ├── rate_limit.py  # Redis rate limiting
│   │   │   │   ├── text_cleaner.py # Response cleaning
│   │   │   │   └── telemetry.py   # OpenTelemetry
│   │   │   ├── db/                # Database
│   │   │   │   ├── session.py     # Async sessions
│   │   │   │   └── init_db.py     # DB initialization
│   │   │   ├── models/            # SQLAlchemy models
│   │   │   │   ├── user.py
│   │   │   │   ├── conversation.py
│   │   │   │   ├── message.py
│   │   │   │   └── router_decision.py
│   │   │   ├── schemas/           # Pydantic schemas
│   │   │   │   ├── chat.py
│   │   │   │   ├── conversation.py
│   │   │   │   └── auth.py
│   │   │   ├── services/          # Business logic
│   │   │   │   ├── llm_client.py  # OpenRouter wrapper
│   │   │   │   ├── router.py      # Model routing (MoE)
│   │   │   │   ├── safety.py      # Safety checks
│   │   │   │   └── history.py     # Conversation history
│   │   │   └── main.py            # FastAPI app entry
│   │   ├── alembic/               # Database migrations
│   │   ├── tests/                 # Test suite
│   │   └── pyproject.toml         # Python dependencies
│   │
│   └── frontend/                  # React Frontend
│       ├── src/
│       │   ├── components/        # Reusable components
│       │   │   ├── ModelSelector.tsx
│       │   │   ├── LoadingSpinner.tsx
│       │   │   └── Logo.tsx
│       │   ├── pages/             # Page components
│       │   │   ├── ChatPage.tsx
│       │   │   ├── LoginPage.tsx
│       │   │   └── SignupPage.tsx
│       │   ├── features/          # Feature modules
│       │   │   ├── auth/
│       │   │   ├── chat/
│       │   │   └── profile/
│       │   ├── lib/               # Utilities
│       │   │   ├── api.ts         # API client
│       │   │   ├── firebase.ts    # Firebase config
│       │   │   └── auth-dev.ts    # Dev auth
│       │   └── App.tsx            # Main app
│       └── package.json
│
├── infra/
│   └── docker-compose.dev.yml     # Local dev services
│
└── test_integrations.py           # Integration test script
```

---

## 🔧 Technology Stack

### Backend

| Component | Technology | Purpose |
|-----------|------------|---------|
| Framework | FastAPI | Async REST API |
| Language | Python 3.11 | Backend logic |
| Database | PostgreSQL 15 | Persistent storage |
| Cache | Redis 7 | Rate limiting, sessions |
| Auth | Firebase Admin | JWT verification |
| ORM | SQLAlchemy 2.0 (async) | Database operations |
| LLM | OpenRouter API | Model inference |
| Observability | OpenTelemetry | Tracing & metrics |

### Frontend

| Component | Technology | Purpose |
|-----------|------------|---------|
| Framework | React 18 | UI components |
| Build | Vite 5 | Fast dev/build |
| Language | TypeScript | Type safety |
| Styling | TailwindCSS | Utility CSS |
| UI | shadcn/ui | Component library |
| Auth | Firebase SDK | Client auth |

---

## 🤖 Model Configuration

### Specialist Models (via OpenRouter)

| Model Type | Display Name | Actual Model ID | Use Case |
|------------|--------------|-----------------|----------|
| Theory | Qwen 2.5 | `qwen/qwen-2.5-72b-instruct` | General questions, explanations |
| Math | Qwen 2.5 | `google/gemma-3-12b-it` | Mathematical calculations |
| Code | Seed Coder | `qwen/qwen-2.5-coder-32b-instruct` | Programming tasks |
| Safety | LlamaGuard | `meta-llama/llama-guard-3-8b` | Content moderation |

### Router Logic (Mixture of Experts)

```
User Query → Intent Classification → Select Specialist Model
         ↓
    Keywords Analysis:
    - Code keywords → code-specialist
    - Math keywords → math-specialist  
    - Theory keywords → theory-specialist
    - Default → theory-specialist
```

### Model Identity Feature

When users ask "What model are you?" or similar questions, the system responds based on the selected model:

| Selected Model | Response |
|----------------|----------|
| Theory Specialist | "I am Qwen 2.5..." |
| Math Specialist | "I am Qwen 2.5..." |
| Code Specialist | "I am Seed Coder..." |
| Auto | "I am Qwen 2.5..." |

---

## 🔐 Security Implementation

| Feature | Implementation |
|---------|----------------|
| Authentication | Firebase JWT tokens |
| Authorization | Per-request token verification |
| CORS | Whitelisted frontend origins |
| Rate Limiting | Redis-based (60/user, 10/IP per minute) |
| Headers | X-Frame-Options, CSP, HSTS |
| Input Validation | Pydantic schemas |
| SQL Injection | SQLAlchemy ORM parameterized queries |
| Content Safety | Input/output safety checks |

---

## 📡 API Endpoints

### Authentication

```
GET  /api/v1/auth/me              # Get current user
```

### Chat

```
POST /api/v1/chat/send            # Send message (sync)
POST /api/v1/chat/send-stream     # Send message (streaming SSE)
```

### Conversations

```
GET    /api/v1/conversations           # List all
POST   /api/v1/conversations           # Create new
GET    /api/v1/conversations/{id}      # Get with messages
PATCH  /api/v1/conversations/{id}      # Update
DELETE /api/v1/conversations/{id}      # Delete
```

### Health & Admin

```
GET /api/v1/healthz              # Health check
GET /api/v1/admin/metrics        # Prometheus metrics
GET /api/v1/admin/config         # Config (admin only)
```

---

## 💾 Database Schema

```
┌──────────────────┐
│      users       │
├──────────────────┤
│ id (UUID, PK)    │
│ firebase_uid     │
│ email            │
│ display_name     │
│ photo_url        │
│ is_admin         │
│ created_at       │
│ updated_at       │
└────────┬─────────┘
         │ 1:N
         ▼
┌──────────────────┐
│  conversations   │
├──────────────────┤
│ id (UUID, PK)    │
│ user_id (FK)     │
│ title            │
│ created_at       │
│ updated_at       │
└────────┬─────────┘
         │ 1:N
         ▼
┌──────────────────┐      ┌────────────────────┐
│    messages      │      │  router_decisions  │
├──────────────────┤      ├────────────────────┤
│ id (UUID, PK)    │──1:1─│ id (UUID, PK)      │
│ conversation_id  │      │ message_id (FK)    │
│ role (enum)      │      │ strategy           │
│ content          │      │ selected_endpoint  │
│ tokens_in        │      │ confidence         │
│ tokens_out       │      │ reasons (JSON)     │
│ latency_ms       │      │ created_at         │
│ safety_labels    │      └────────────────────┘
│ created_at       │
└──────────────────┘
```

---

## 🚀 Features Implemented

### ✅ Completed

- [x] User authentication (Firebase Auth + dev mode)
- [x] ChatGPT-like chat interface
- [x] Conversation CRUD operations
- [x] Message history with context
- [x] Real-time streaming responses (SSE)
- [x] Model routing (theory/code/math specialists)
- [x] Model identity responses ("What model are you?")
- [x] Safety layer (placeholder for Llama Guard)
- [x] Rate limiting (Redis-based)
- [x] Response cleaning (removes thinking tags, markdown)
- [x] CORS configuration (multi-port support)
- [x] Health checks (DB, Redis)
- [x] OpenTelemetry integration
- [x] Prometheus metrics endpoint
- [x] Integration test suite

### 🔄 Placeholders (Ready for Implementation)

- [ ] Llama Guard safety integration (currently mock)
- [ ] Production Firebase configuration
- [ ] Cloud SQL connection
- [ ] Cloud Memorystore connection

---

## ⚙️ Configuration

### Environment Variables (Backend)

```env
# General
ENV=development
FRONTEND_ORIGIN=http://localhost:5173,http://localhost:5174

# Database
POSTGRES_HOST=localhost
POSTGRES_PORT=5433
POSTGRES_DB=chatapp
POSTGRES_USER=chatapp
POSTGRES_PASSWORD=chatapp

# Redis
REDIS_URL=redis://localhost:6379/0

# OpenRouter (LLM)
INFERENCE_BASE_URL=https://openrouter.ai/api/v1
INFERENCE_API_KEY=sk-or-xxx  # Your OpenRouter key

# Models (optional overrides)
MODEL_THEORY=qwen/qwen-2.5-72b-instruct
MODEL_CODE=qwen/qwen-2.5-coder-32b-instruct
MODEL_MATH=google/gemma-3-12b-it

# Firebase
FIREBASE_PROJECT_ID=your-project
```

### Token Limits

| Setting | Default | Min | Max |
|---------|---------|-----|-----|
| max_tokens | 2,048 | 1 | 8,192 |
| temperature | 0.7 | 0.0 | 2.0 |
| message length | - | 1 | 10,000 chars |

---

## 🧪 Testing

### Run Integration Tests

```bash
python test_integrations.py
```

### Test Coverage

- ✅ Health check (DB, Redis)
- ✅ Authentication (dev mode)
- ✅ Conversation CRUD
- ✅ Chat send endpoint
- ✅ Streaming endpoint
- ✅ Router classification

---

## 🏃 Running the Application

### Start Services

```bash
# 1. Start infrastructure
docker-compose -f infra/docker-compose.dev.yml up -d postgres redis

# 2. Start backend
cd app/backend
python -m uvicorn app.main:app --host 0.0.0.0 --port 8000

# 3. Start frontend
cd app/frontend
npm run dev
```

### Access Points

| Service | URL |
|---------|-----|
| Frontend | http://localhost:5173 or :5174 |
| Backend API | http://localhost:8000 |
| API Docs | http://localhost:8000/docs |
| Health Check | http://localhost:8000/api/v1/healthz |

---

## 📊 Current Status

```
┌────────────────────────────────────────────────────────┐
│                  INTEGRATION STATUS                     │
├────────────────────────────────────────────────────────┤
│  ✅ Backend Server          Running on :8000           │
│  ✅ Frontend Server         Running on :5174           │
│  ✅ PostgreSQL              Running on :5433           │
│  ✅ Redis                   Running on :6379           │
│  ✅ OpenRouter API          Connected                  │
│  ✅ Firebase Auth           Dev mode enabled           │
│  ✅ CORS                    Configured                 │
│  ✅ All Integration Tests   Passing                    │
└────────────────────────────────────────────────────────┘
```

---

## 📝 Chat Request/Response Format

### Request

```json
{
  "message": "Your question here",
  "conversation_id": "uuid-optional",
  "meta": {
    "model": "auto",
    "temperature": 0.7,
    "max_tokens": 2048
  }
}
```

### Response

```json
{
  "assistant_message": "Response from the model",
  "conversation_id": "uuid",
  "message_id": "uuid",
  "router": {
    "strategy": "moe",
    "endpoint": "theory-specialist",
    "confidence": 0.75
  },
  "safety": {
    "input": {"allowed": true},
    "output": {"allowed": true}
  }
}
```

---

## 🔄 Message Flow

```
1. User sends message
         ↓
2. Firebase token verification
         ↓
3. Rate limit check (Redis)
         ↓
4. Safety-In check (input validation)
         ↓
5. Router decides specialist model
         ↓
6. Check for model identity question
         ↓
7. Call OpenRouter API (or return identity)
         ↓
8. Clean response (remove thinking tags)
         ↓
9. Safety-Out check (output validation)
         ↓
10. Save to database
         ↓
11. Return response to user
```

---

## 📄 License

MIT

---

*Last Updated: December 5, 2025*

