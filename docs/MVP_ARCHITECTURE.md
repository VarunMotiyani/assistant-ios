---
name: MVP Architecture
description: Complete MVP specification for personal JARVIS using Supabase + Render + Together.ai
version: 1.0
date: 2026-04-05
---

# JARVIS MVP Architecture

Personal JARVIS built for yourself. Zero ops, maximum features, minimal cost.

## Tech Stack

```
Frontend:       iOS app (Swift/SwiftUI)
Backend:        FastAPI (Python) on Render.com
Database:       Supabase (PostgreSQL + pgvector)
LLM:            Together.ai or Groq (open-source models)
Auth:           Supabase Auth
Realtime:       Supabase Realtime
Cost:           $2-5/month
```

---

## Architecture Diagram

```
┌─────────────────────┐
│   iOS App (Swift)   │
│  Command Center UI  │
└──────────┬──────────┘
           │ HTTPS
           ▼
┌───────────────────────────────────────┐
│   Render.com (FastAPI Backend)        │
│   https://jarvis-backend.onrender.com │
│                                       │
│  ├─ Orchestrator (agent router)      │
│  ├─ 7 Agents (LifeAdmin, etc)        │
│  ├─ Integration handlers             │
│  └─ API endpoints                    │
└───────────┬───────────────────────────┘
            │
    ┌───────┴─────────┬──────────┬──────────┐
    ▼                 ▼          ▼          ▼
┌─────────────┐ ┌─────────┐ ┌──────────┐ ┌──────────┐
│ Supabase DB │ │Together │ │Google    │ │Notion    │
│ PostgreSQL  │ │.ai LLM  │ │Calendar  │ │API       │
│ + pgvector  │ │         │ │OAuth     │ │OAuth     │
└─────────────┘ └─────────┘ └──────────┘ └──────────┘
    │                          │          │
    └──────────┬───────────────┴──────────┘
               │
    ┌──────────┴──────────┬──────────┬──────────┐
    ▼                     ▼          ▼          ▼
 ┌───────┐         ┌──────────┐ ┌────────┐ ┌─────────┐
 │Events │         │Users     │ │Tasks   │ │Semantic │
 │Log    │         │Profile   │ │Memory  │ │Memory   │
 └───────┘         └──────────┘ └────────┘ └─────────┘
```

---

## MVP Scope (Phase 1: 2 Weeks)

### What's Included

✓ **Dashboard (Command Center)**
- Daily briefing (merged from all agents)
- Critical alerts
- Quick action buttons
- Real-time updates via WebSocket

✓ **6 Core Agents**
- **LifeAdmin**: Tasks, calendar, schedule optimization
- **Wellness**: Health tracking, recovery score
- **Finance**: Portfolio summary, alerts
- **Content**: Top stories, news
- **Relationships**: Contacts, reach-outs
- **Critic**: Evaluation, decision-making

✓ **3 Essential Integrations**
- Google Calendar (read events)
- Notion (read/write tasks)
- Apple HealthKit (via iOS app)

✓ **Memory System**
- Short-term cache (Redis via Supabase)
- Long-term storage (PostgreSQL)
- Semantic search (pgvector)

### What's Deferred (Phase 2+)

- Email integration
- Instagram/WhatsApp
- Groww/stock portfolio
- Apple Calendar
- Shortcuts/Automation
- Productivity Agent details
- Multi-user support

---

## Deployment: Supabase + Render Setup

### Step 1: Supabase (5 minutes)

```bash
# 1. Go to https://supabase.com
# 2. Sign up with GitHub
# 3. Create new project (any region)
# 4. Go to Settings → Database → Connection Pooling
# Copy CONNECTION STRING (looks like):
# postgresql://postgres.xxxxx:password@db.supabase.co:5432/postgres

# 5. Enable pgvector extension
# Go to SQL Editor → paste:
CREATE EXTENSION IF NOT EXISTS vector;
```

### Step 2: Backend Code (Your Repo)

```
jarvis/
├── backend/
│   ├── main.py              # FastAPI app
│   ├── config.py            # Database config
│   ├── models.py            # Pydantic models
│   ├── database.py          # SQLAlchemy setup
│   ├── agents/              # 7 agent implementations
│   │   ├── __init__.py
│   │   ├── base.py          # Base Agent class
│   │   ├── life_admin.py
│   │   ├── wellness.py
│   │   ├── finance.py
│   │   ├── content.py
│   │   ├── relationships.py
│   │   └── critic.py
│   ├── integrations/        # API client wrappers
│   │   ├── google_calendar.py
│   │   ├── notion.py
│   │   └── health_kit.py
│   ├── memory/              # Memory manager
│   │   └── memory_manager.py
│   └── orchestrator/        # Agent orchestrator
│       └── orchestrator.py
├── requirements.txt
├── Dockerfile
└── render.yaml
```

### Step 3: Render Deployment (3 minutes)

```bash
# 1. Push code to GitHub
git add .
git commit -m "MVP JARVIS"
git push origin main

# 2. Go to https://render.com
# 3. Click "New Web Service"
# 4. Connect GitHub account
# 5. Select your repo
# 6. Fill in settings:
#    Name: jarvis-backend
#    Branch: main
#    Runtime: Python 3.11
#    Build command: pip install -r requirements.txt
#    Start command: uvicorn backend.main:app --host 0.0.0.0 --port $PORT

# 7. Add Environment Variables:
DATABASE_URL=postgresql://postgres.xxxxx:password@db.supabase.co:5432/postgres
TOGETHER_API_KEY=xxx
SECRET_KEY=<random-32-char-string>
GOOGLE_CLIENT_ID=xxx
GOOGLE_CLIENT_SECRET=xxx
NOTION_API_KEY=xxx

# 8. Deploy
# Render automatically builds and deploys from GitHub
# Your backend is live at: https://jarvis-backend.onrender.com
```

### Step 4: iOS App Configuration

```swift
// AppDelegate.swift or main config
struct APIConfig {
    static let baseURL = URL(string: "https://jarvis-backend.onrender.com")!
    static let wsURL = URL(string: "wss://jarvis-backend.onrender.com/ws")!
}
```

---

## Database Schema (MVP)

```sql
-- Users
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email TEXT UNIQUE NOT NULL,
    name TEXT,
    created_at TIMESTAMP DEFAULT now(),
    settings JSONB DEFAULT '{}'
);

-- Events Log (core data)
CREATE TABLE events_log (
    id BIGSERIAL PRIMARY KEY,
    user_id UUID NOT NULL REFERENCES users(id),
    event_type VARCHAR(50),  -- "task_created", "decision_made", "briefing_generated"
    payload JSONB,
    created_at TIMESTAMP DEFAULT now(),
    INDEX (user_id, created_at)
);

-- Semantic Memory (for Critic agent)
CREATE TABLE semantic_memory (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id),
    content TEXT NOT NULL,
    embedding vector(1536),  -- pgvector
    source VARCHAR(50),      -- "critic_session", "daily_briefing"
    created_at TIMESTAMP DEFAULT now(),
    INDEX ON semantic_memory USING ivfflat (embedding vector_cosine_ops)
);

-- Tasks (from Notion)
CREATE TABLE tasks (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id),
    notion_page_id TEXT,     -- Sync with Notion
    title TEXT NOT NULL,
    status VARCHAR(50),      -- "not_started", "in_progress", "completed"
    due_date DATE,
    priority VARCHAR(20),    -- "high", "medium", "low"
    category TEXT,
    created_at TIMESTAMP DEFAULT now(),
    updated_at TIMESTAMP DEFAULT now(),
    INDEX (user_id, status)
);

-- Calendar Events (from Google)
CREATE TABLE calendar_events (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id),
    google_event_id TEXT UNIQUE,  -- Sync with Google
    title TEXT NOT NULL,
    start_time TIMESTAMP NOT NULL,
    end_time TIMESTAMP NOT NULL,
    description TEXT,
    created_at TIMESTAMP DEFAULT now(),
    updated_at TIMESTAMP DEFAULT now(),
    INDEX (user_id, start_time)
);

-- Health/Wellness Data
CREATE TABLE wellness_snapshots (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id),
    date DATE NOT NULL,
    sleep_minutes INT,
    deep_sleep_minutes INT,
    workout_count INT,
    workout_minutes INT,
    recovery_score INT,
    hrv INT,
    resting_heart_rate INT,
    created_at TIMESTAMP DEFAULT now(),
    UNIQUE (user_id, date),
    INDEX (user_id, date)
);

-- Contacts (for Relationships agent)
CREATE TABLE contacts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id),
    name TEXT NOT NULL,
    email TEXT,
    phone TEXT,
    last_contacted TIMESTAMP,
    interaction_count INT DEFAULT 0,
    notes TEXT,
    created_at TIMESTAMP DEFAULT now(),
    INDEX (user_id, last_contacted)
);

-- OAuth Tokens (encrypted)
CREATE TABLE oauth_tokens (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id),
    service VARCHAR(50),  -- "google_calendar", "notion"
    access_token TEXT,    -- ENCRYPT THIS
    refresh_token TEXT,   -- ENCRYPT THIS
    expires_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT now(),
    UNIQUE (user_id, service),
    INDEX (user_id)
);

-- Financial Data (placeholder for Groww)
CREATE TABLE portfolio_holdings (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id),
    symbol TEXT NOT NULL,
    quantity DECIMAL(10, 2),
    current_price DECIMAL(10, 2),
    current_value DECIMAL(15, 2),
    day_change_percent DECIMAL(5, 2),
    updated_at TIMESTAMP DEFAULT now(),
    INDEX (user_id)
);

-- Daily Briefing Cache
CREATE TABLE briefing_cache (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id),
    date DATE NOT NULL,
    briefing_data JSONB,  -- Cached full briefing
    created_at TIMESTAMP DEFAULT now(),
    UNIQUE (user_id, date),
    INDEX (user_id, date)
);
```

---

## API Endpoints (MVP)

### Authentication

```
POST /auth/signup
  Body: { email, password, name }
  Response: { user_id, token }

POST /auth/login
  Body: { email, password }
  Response: { user_id, token }

POST /auth/google/callback
  Body: { code }
  Response: { token, redirects_to: "/oauth/select" }
```

### Main Dashboard

```
GET /briefing/today
  Headers: { Authorization: "Bearer token" }
  Response: {
    date: "2026-04-05",
    life_admin: { inbox: [], top_tasks: [] },
    wellness: { recovery_score: 78, health_trend: "good" },
    finance: { portfolio_value: 125000, day_change: 2.5 },
    content: { top_stories: [] },
    relationships: { reach_outs: [] },
    generated_at: "2026-04-05T08:00:00Z"
  }
```

### Tasks

```
GET /tasks
  Query: { status?, due_date_from?, due_date_to? }
  Response: [ { id, title, status, due_date, priority, category } ]

POST /tasks
  Body: { title, due_date, priority, category }
  Response: { id, ... }

PATCH /tasks/{id}
  Body: { status?, title?, due_date? }
  Response: { id, ... }
```

### Calendar

```
GET /calendar/events
  Query: { from_date, to_date }
  Response: [ { id, title, start_time, end_time, description } ]

POST /calendar/events
  Body: { title, start_time, end_time, description }
  Response: { id, ... }
```

### Health/Wellness

```
GET /health/snapshot/today
  Response: { sleep_minutes, recovery_score, workouts, ... }

POST /health/sync
  Body: { date, sleep_minutes, workouts, hrv, ... }
  Response: { ok: true }
```

### Integrations

```
GET /integrations/status
  Response: { google_calendar: "connected", notion: "connected", ... }

POST /integrations/google-calendar/connect
  Body: { oauth_code }
  Response: { ok: true }

POST /integrations/notion/connect
  Body: { oauth_code }
  Response: { ok: true }
```

### Critic Evaluation

```
POST /critic/evaluate
  Body: { prompt }
  Response: {
    verdict: "string",
    reasoning: "string",
    assumptions: ["array"],
    action_items: ["array"]
  }
```

### Memory/Search

```
POST /memory/search
  Body: { query, limit?: 5 }
  Response: [ { content, source, similarity } ]

POST /memory/semantic-search
  Body: { query, limit?: 5 }
  Response: [ { content, source, similarity } ]
```

### Realtime Updates

```
WS /ws
  Headers: { Authorization: "Bearer token" }
  
  Subscribe to real-time updates:
  message: {
    type: "briefing_updated" | "task_created" | "agent_response",
    payload: { ... }
  }
```

---

## MVP Agent Implementations

### LifeAdmin Agent

```python
class LifeAdminAgent:
    async def get_daily_inbox(self, user_id: str) -> Dict:
        """Today's critical tasks + events"""
        # Query tasks (status != "completed", due_date = today)
        # Query calendar (today's events)
        # Rank by priority/urgency
        return {
            "urgent_tasks": [...],
            "calendar_today": [...],
            "estimated_free_time": "4.5 hours",
            "recommendations": ["Complete X before 2pm", ...]
        }
    
    async def get_weekly_schedule(self, user_id: str) -> Dict:
        """Optimize schedule"""
        # AI suggests optimal times to schedule tasks based on
        # calendar, recovery score, productivity patterns
        pass
```

### Wellness Agent

```python
class WellnessAgent:
    async def get_recovery_score(self, user_id: str) -> Dict:
        """Calculate recovery score (0-100)"""
        # Based on: sleep, HRV, workouts, stress patterns
        # From HealthKit data synced via iOS app
        return {
            "score": 78,
            "factors": {
                "sleep": 85,
                "recovery": 72,
                "trend": "improving"
            },
            "recommendation": "Light activity recommended"
        }
```

### Finance Agent

```python
class FinanceAgent:
    async def get_portfolio_summary(self, user_id: str) -> Dict:
        """Portfolio + watchlist"""
        # Will integrate with Groww API later
        return {
            "total_value": 125000,
            "day_change": 2.5,
            "day_change_percent": 2.1,
            "top_performers": [...],
            "alerts": ["Stock X dropped 5%"]
        }
```

### Content Agent

```python
class ContentAgent:
    async def get_top_stories(self, user_id: str) -> Dict:
        """Personalized news feed"""
        # Fetch from: HackerNews, TechCrunch, Verge, arXiv
        # Filter by user interests (tech, startups, AI, etc)
        return {
            "stories": [
                {
                    "title": "...",
                    "source": "hackernews",
                    "url": "...",
                    "summary": "..."
                }
            ]
        }
```

### Relationships Agent

```python
class RelationshipsAgent:
    async def get_reach_outs(self, user_id: str) -> Dict:
        """Contacts who haven't heard from you"""
        # Query contacts table
        # Calculate days since last contact
        # Prioritize by importance
        return {
            "reach_outs": [
                {
                    "name": "Alice",
                    "last_contacted": "45 days ago",
                    "interaction_count": 12,
                    "suggested_action": "Schedule coffee"
                }
            ]
        }
```

### Critic Agent

```python
class CriticAgent:
    async def evaluate(
        self,
        user_id: str,
        prompt: str,
        context: Dict = None
    ) -> Dict:
        """Honest evaluation of ideas"""
        # Call Together.ai LLM
        # Inject context from other agents
        # Return verdict + reasoning
        
        # Example: "Should I launch my course?"
        # Gets: wellness (busy/not busy), finance (money constraints),
        #       productivity (track record), relationships (time for community)
        
        return {
            "verdict": "Yes, but with caveats",
            "reasoning": "...",
            "assumptions": ["Assumption 1", "Assumption 2"],
            "action_items": ["First step", "Second step"]
        }
```

---

## Integration Priorities (MVP vs Later)

### Phase 1 (MVP - Week 1-2)

**Must Have:**
- Google Calendar (read events)
- Notion (read/write tasks)
- Apple HealthKit (sync via iOS app)

### Phase 2 (Weeks 3-4)

- Email (Gmail read-only)
- Groww portfolio API
- Apple Reminders sync
- Spotify (listening habits)

### Phase 3 (Later - Optional)

- Instagram API (limitations exist)
- WhatsApp (API access limited)
- Shortcuts automation
- Browser history
- Twitter/X API

---

## Environment Variables (Render)

```bash
# Database
DATABASE_URL=postgresql://postgres.xxxxx:password@db.supabase.co:5432/postgres

# LLM
TOGETHER_API_KEY=sk-xxx
TOGETHER_MODEL=mistralai/Mixtral-8x7B-Instruct-v0.1

# Google OAuth
GOOGLE_CLIENT_ID=xxx.apps.googleusercontent.com
GOOGLE_CLIENT_SECRET=xxx
GOOGLE_REDIRECT_URI=https://jarvis-backend.onrender.com/integrations/google/callback

# Notion
NOTION_API_KEY=secret_xxx
NOTION_ICON_EMOJI=🤖

# Security
SECRET_KEY=<generate-with: python -c "import secrets; print(secrets.token_hex(32))">
JWT_ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=1440

# Environment
ENVIRONMENT=production
DEBUG=false
LOG_LEVEL=info
```

---

## Deployment Checklist

- [ ] Create Supabase project
- [ ] Enable pgvector extension
- [ ] Create all tables (SQL schema)
- [ ] Create GitHub repo with backend code
- [ ] Sign up for Together.ai (free tier)
- [ ] Get Google OAuth credentials (Google Cloud)
- [ ] Get Notion integration token
- [ ] Deploy to Render
- [ ] Set environment variables on Render
- [ ] Test API endpoints
- [ ] Connect iOS app to backend URL
- [ ] Test OAuth flows

---

## Monthly Cost

```
Supabase:        FREE (500MB storage, generous limits)
Render:          FREE (spins down after 15min inactivity)
Together.ai:     $2-5/mo (if you use > free tier)
Domain:          $0 (use Render subdomain) or $1/year
─────────────────────────────────────────────
TOTAL:           $0-5/month
```

When you scale:
```
Supabase Pro:    $25/mo (8GB)
Render:          $7/mo (always-on)
Together.ai:     $5-10/mo
─────────────────────────────────────────────
TOTAL:           $37-42/month
```

---

## Next Steps

1. **Today**: Set up Supabase + Render
2. **Tomorrow**: Implement backend with FastAPI
3. **This Week**: Implement 3 core integrations (Google, Notion, HealthKit)
4. **Next Week**: Build iOS UI (Command Center, Tasks, Calendar)
5. **Following Week**: Connect everything together

---

## Files to Create

```
backend/main.py              # FastAPI app entry point
backend/config.py            # Database + env config
backend/models.py            # Pydantic request/response models
backend/database.py          # SQLAlchemy ORM models
backend/agents/base.py       # Base Agent class
backend/agents/life_admin.py # Implemented
backend/agents/wellness.py   # Implemented
backend/agents/finance.py    # Implemented
backend/agents/content.py    # Implemented
backend/agents/relationships.py # Implemented
backend/agents/critic.py     # Implemented
backend/integrations/google_calendar.py
backend/integrations/notion.py
backend/integrations/health_kit.py
backend/memory/memory_manager.py
backend/orchestrator/orchestrator.py
requirements.txt
Dockerfile
render.yaml
```

---

Ready to start building? Start with `backend/main.py` and `backend/config.py`?
