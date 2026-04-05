# JARVIS Backend API Specification

## Overview

The backend is a **FastAPI** Python service that exposes REST endpoints + WebSocket for real-time updates. All business logic, LLM calls, and agent orchestration happen here.

**Base URL:** `https://api.jarvis.yourdomain.com/v1`

---

## Authentication

All requests require a **Bearer JWT token** in the `Authorization` header.

```
Authorization: Bearer <jwt_token>
```

**Token generation:**
- User authenticates (email + magic link / OAuth)
- Backend returns JWT with 24h expiry
- Frontend stores in Keychain (iOS) or localStorage (web)
- Frontend includes in every request

**Token structure:**
```json
{
  "sub": "user_uuid",
  "email": "user@example.com",
  "exp": 1234567890,
  "iat": 1234567800
}
```

---

## Endpoints by Domain

## 1. Briefing API

### GET `/briefing/today`

Fetch today's daily briefing (all agents' summaries merged).

**Response:**
```json
{
  "date": "2026-04-05",
  "generated_at": "2026-04-05T06:45:00Z",
  
  "life_admin": {
    "top_3_tasks": [
      {
        "id": "uuid",
        "title": "Close Q1 billing",
        "category": "startup",
        "priority": "critical",
        "due_today": true
      }
    ],
    "critical_alerts": [
      {
        "type": "bill_due",
        "name": "AWS subscription",
        "amount_inr": 2500,
        "days_until_due": 2
      }
    ]
  },
  
  "wellness": {
    "recovery_score": 62,
    "sleep_last_night": {
      "duration_hours": 7.2,
      "efficiency": 86,
      "deep_minutes": 90,
      "rem_minutes": 110
    },
    "workout_recommendation": "moderate",
    "coaching_note": "Good recovery. Push hard today but respect fatigue.",
    "burnout_risk": "low"
  },
  
  "finance": {
    "portfolio_value": "₹5,012,500",
    "today_delta": {
      "amount": "+₹12,500",
      "percent": "+1.2%"
    },
    "top_movers": [
      {
        "symbol": "HDFCBANK",
        "change_percent": "+2.1%"
      }
    ],
    "savings_rate_this_month": 38
  },
  
  "content": {
    "top_3_stories": [
      {
        "id": "feed_item_uuid",
        "title": "OpenAI releases GPT-4.5 with multimodal reasoning",
        "source": "techcrunch",
        "summary": "2-line hook",
        "relevance_score": 9.2,
        "idea_card_available": true
      }
    ]
  },
  
  "relationships": {
    "reach_outs_today": [
      {
        "contact_id": "uuid",
        "name": "Rahul",
        "relationship_type": "friend",
        "days_since_contact": 18,
        "suggested_message": "Hey! Been a while. How's the new project going?"
      }
    ],
    "upcoming_moments": [
      {
        "contact_id": "uuid",
        "name": "Mom",
        "event": "birthday",
        "days_until": 5
      }
    ]
  }
}
```

---

## 2. Agent Query API

### POST `/agents/{agent_name}/query`

Ask an agent a question and get back a response.

**Request:**
```json
{
  "query_type": "string", // e.g., "recovery_score_today", "contact_urgency_scores"
  "context": {} // optional additional context
}
```

**Example: Ask Wellness about recovery**
```bash
POST /agents/wellness/query
{
  "query_type": "recovery_score_today"
}
```

**Response:**
```json
{
  "recovery_score": 62,
  "sleep": 7.2,
  "hrv": 45,
  "resting_hr": 58,
  "mood": 7,
  "reasoning": "Good sleep, normal HRV, stable mood. Ready for moderate work.",
  "timestamp": "2026-04-05T08:15:00Z"
}
```

**Supported queries by agent:**

**Wellness:**
- `recovery_score_today` → recovery score + factors
- `workout_recommendation` → light/moderate/heavy/rest
- `burnout_risk` → low/medium/high with reasoning

**Finance:**
- `portfolio_value` → current portfolio value
- `cashflow_summary` → monthly income vs expenses
- `emergency_fund_coverage` → months of runway

**LifeAdmin:**
- `daily_inbox` → top 3 tasks due today
- `calendar_load_today` → meeting hours, free time blocks
- `critical_alerts` → overdue items, bills due soon

**Relationships:**
- `reach_outs_today` → who to contact today
- `upcoming_moments` → birthdays/anniversaries next 14 days

**Content:**
- `top_stories` → top N items from feed
- `idea_card` → full idea card for a story

**Critic:**
- (Critic is queried via `/agents/critic/evaluate`, not `/agents/critic/query`)

---

## 3. Critic Agent API

### POST `/agents/critic/evaluate`

Submit an idea/plan/decision for brutal critique.

**Request:**
```json
{
  "prompt": "Should I build an AI course for founders?",
  "category": "startup_idea", // startup_idea, plan, decision, question
  "inject_context": true, // include wellness/finance/calendar context?
  "voice_transcript": null // optional: transcribed speech
}
```

**Response:**
```json
{
  "session_id": "uuid",
  "timestamp": "2026-04-05T09:30:00Z",
  
  "verdict": "yellow", // red = don't do it, yellow = risky, green = go
  "verdict_summary": "Risky timing given burnout risk, but idea has merit.",
  
  "core_critique": [
    "1. Recovery score is low (62). Starting intense project now risks burnout.",
    "2. You assume 3 months runway but have 8. You're overestimating runway comfort.",
    "3. AI course market is oversaturated. What's your differentiation?"
  ],
  
  "assumptions": [
    "Assumes course will sell (validation missing)",
    "Assumes you can sustain 60h/week for 3 months (historically false for you)",
    "Assumes income will start in month 1 (unrealistic)"
  ],
  
  "missing_info": [
    "Do you have a list of potential students?",
    "What's your launch timeline? (vague)",
    "Have you validated course topics with real founders?"
  ],
  
  "if_you_must_proceed": [
    "1. Wait 2 weeks for recovery to stabilize.",
    "2. Validate course ideas with 5+ target customers first.",
    "3. Commit max 20h/week, not full-time."
  ],
  
  "personal_context": "You're already managing startup anxiety. Adding course creation now is compounding risk. You thrive when focused on one thing."
}
```

### GET `/agents/critic/sessions`

Fetch all past critique sessions.

**Query params:**
- `limit` → max 50 sessions (default: 20)
- `offset` → pagination
- `category` → filter by startup_idea, plan, decision, etc.

**Response:**
```json
{
  "sessions": [
    {
      "id": "uuid",
      "timestamp": "2026-04-05T09:30:00Z",
      "prompt_preview": "Should I build an AI course?",
      "category": "startup_idea",
      "verdict": "yellow"
    }
  ],
  "total": 42,
  "offset": 0
}
```

### GET `/agents/critic/sessions/{session_id}`

Fetch full details of a past critique.

---

## 4. Health / Wellness API

### POST `/health/mood`

Log a mood check-in.

**Request:**
```json
{
  "score": 7,  // 1-10
  "note": "Good day, productive",
  "energy_level": 8
}
```

**Response:**
```json
{
  "id": "uuid",
  "logged_at": "2026-04-05T09:15:00Z",
  "ok": true
}
```

### GET `/health/snapshots/latest`

Fetch latest health snapshot.

**Response:**
```json
{
  "date": "2026-04-05",
  "sleep_duration_minutes": 432,
  "sleep_efficiency_pct": 86,
  "sleep_deep_minutes": 90,
  "sleep_rem_minutes": 110,
  "hrv_ms": 45,
  "resting_hr_bpm": 58,
  "steps": 8234,
  "active_calories": 320,
  "stand_hours": 8,
  "body_weight_kg": 82.5,
  "recovery_score": 62,
  "workload_score": 65,
  "synced_at": "2026-04-05T08:15:00Z"
}
```

### GET `/health/snapshots/range`

Fetch health snapshots for a date range (for charts).

**Query params:**
- `start_date` → YYYY-MM-DD
- `end_date` → YYYY-MM-DD

**Response:**
```json
{
  "snapshots": [
    { "date": "2026-04-05", "recovery_score": 62, "sleep": 7.2, ... },
    { "date": "2026-04-04", "recovery_score": 58, "sleep": 6.8, ... }
  ]
}
```

### POST `/health/supplement`

Log supplement intake.

**Request:**
```json
{
  "supplement": "creatine", // creatine, whey_post, whey_second
  "date": "2026-04-05"
}
```

---

## 5. Finance API

### GET `/finance/holdings`

Fetch all holdings with current prices and P&L.

**Response:**
```json
{
  "holdings": [
    {
      "id": "uuid",
      "symbol": "HDFCBANK",
      "name": "HDFC Bank",
      "asset_type": "equity",
      "quantity": 50,
      "avg_cost_price": 1850,
      "current_price": 1920,
      "current_value": 96000,
      "unrealized_pnl": 3500,
      "unrealized_pnl_pct": 3.8
    }
  ],
  "total_value": "₹5,012,500",
  "total_pnl": "₹450,000",
  "total_pnl_pct": 9.8
}
```

### GET `/finance/net-worth`

Fetch net worth with trend.

**Response:**
```json
{
  "net_worth": "₹5,500,000",
  "portfolio_value": "₹5,012,500",
  "cash": "₹487,500",
  "trend_30_days": [
    { "date": "2026-03-06", "value": 5250000 },
    { "date": "2026-03-07", "value": 5280000 }
  ],
  "trend_direction": "up"
}
```

### GET `/finance/cashflow`

Monthly cashflow summary.

**Response:**
```json
{
  "month": "2026-03",
  "income": 250000,
  "expenses": 155000,
  "savings": 95000,
  "savings_rate_pct": 38,
  "expenses_by_category": {
    "food": 25000,
    "rent": 50000,
    "utilities": 12000,
    "shopping": 18000,
    "other": 50000
  }
}
```

---

## 6. Content API

### GET `/content/feed/today`

Fetch today's curated feed.

**Response:**
```json
{
  "items": [
    {
      "id": "uuid",
      "title": "OpenAI releases GPT-4.5",
      "source": "techcrunch",
      "url": "https://...",
      "summary": "2-line summary of the story",
      "relevance_score": 9.2,
      "published_at": "2026-04-05T08:00:00Z",
      "idea_card_available": true,
      "tags": ["ai", "llm", "model-release"]
    }
  ]
}
```

### GET `/content/idea-card/{feed_item_id}`

Fetch full idea card for a story.

**Response:**
```json
{
  "feed_item_id": "uuid",
  "title": "OpenAI releases GPT-4.5",
  
  "summary_bullets": [
    "GPT-4.5 introduces faster inference",
    "New reasoning benchmark improvements",
    "Available via API and ChatGPT Plus",
    "Cheaper pricing than GPT-4"
  ],
  
  "why_it_matters": "As an AI-obsessed engineer, this affects your work. Cheaper inference cost improves unit economics for AI products.",
  
  "talking_points": [
    "Faster models = cost reduction for deployed AI",
    "Should you migrate existing projects to 4.5?",
    "Competitive pressure: when does Claude respond?"
  ],
  
  "related_concepts": ["LLM inference", "cost optimization", "model benchmarks"],
  
  "saved": false
}
```

### POST `/content/interact`

Track user interaction with a story (save, like, skip).

**Request:**
```json
{
  "feed_item_id": "uuid",
  "action": "saved" // viewed, saved, liked, skipped
}
```

---

## 7. Relationships / Contacts API

### GET `/contacts`

Fetch all contacts sorted by urgency.

**Query params:**
- `sort_by` → urgency_score (default), name, last_contacted
- `relationship_type` → filter by family, close_friend, professional, etc.

**Response:**
```json
{
  "contacts": [
    {
      "id": "uuid",
      "name": "Rahul",
      "relationship_type": "close_friend",
      "birthday": "1995-06-15",
      "last_contacted": "2026-03-18",
      "contact_frequency": "biweekly",
      "importance": "high",
      "urgency_score": 0.85,
      "days_overdue": 5
    }
  ]
}
```

### GET `/contacts/reach-outs-today`

Fetch today's reach-out recommendations with message drafts.

**Response:**
```json
{
  "contacts": [
    {
      "id": "uuid",
      "name": "Rahul",
      "days_since_contact": 18,
      "suggested_message": "Hey Rahul! Been a while. How's the new project going?"
    }
  ]
}
```

### POST `/contacts/{contact_id}/log-interaction`

Mark that you contacted someone.

**Request:**
```json
{
  "channel": "message", // call, message, email, in_person
  "notes": "Caught up on new project, seems going well"
}
```

### GET `/contacts/upcoming-moments`

Birthdays and anniversaries next 14 days.

**Response:**
```json
{
  "moments": [
    {
      "contact_id": "uuid",
      "name": "Mom",
      "event_type": "birthday",
      "date": "2026-04-10",
      "days_until": 5
    }
  ]
}
```

### POST `/contacts`

Create a new contact.

**Request:**
```json
{
  "name": "Priya",
  "relationship_type": "friend",
  "birthday": "1996-05-20",
  "contact_frequency": "monthly",
  "importance": "medium"
}
```

---

## 8. Tasks / Life Admin API

### GET `/tasks/inbox-today`

Fetch today's tasks.

**Response:**
```json
{
  "tasks": [
    {
      "id": "uuid",
      "title": "Review Q1 metrics",
      "category": "startup",
      "priority": "critical",
      "due_today": true,
      "status": "not_started"
    }
  ]
}
```

### POST `/tasks`

Create a task.

**Request:**
```json
{
  "title": "Launch beta cohort",
  "category": "startup",
  "priority": "high",
  "due_date": "2026-04-15"
}
```

### PATCH `/tasks/{task_id}`

Update a task (mark complete, reschedule, etc.).

**Request:**
```json
{
  "status": "done", // or: not_started, in_progress, done
  "due_date": "2026-04-20" // optional: reschedule
}
```

---

## 9. WebSocket (Real-Time Updates)

### WS `/ws/updates`

Subscribe to real-time updates from the backend.

**Connection:**
```javascript
const ws = new WebSocket('wss://api.jarvis.yourdomain.com/ws/updates?token=<jwt>');
```

**Messages from server:**
```json
{
  "type": "briefing_updated",
  "data": { "date": "2026-04-05", ... }
}
```

```json
{
  "type": "notification",
  "data": { "title": "...", "body": "..." }
}
```

```json
{
  "type": "portfolio_alert",
  "data": { "delta": "-2.5%", "message": "..." }
}
```

---

## 10. Memory / Debug API

### GET `/memory/events`

Fetch event history for debugging.

**Query params:**
- `event_type` → filter
- `limit` → default 50
- `days_back` → default 30

**Response:**
```json
{
  "events": [
    {
      "timestamp": "2026-04-05T08:15:00Z",
      "event_type": "workout_completed",
      "data": { "type": "strength", "duration": 60 }
    }
  ]
}
```

### GET `/memory/export`

Export all user memory (JSON).

---

## Error Responses

All errors return consistent format:

```json
{
  "error": "field_validation_error",
  "message": "Invalid mood score: must be 1-10",
  "status": 400
}
```

**Common status codes:**
- `200` — OK
- `400` — Bad request (validation error)
- `401` — Unauthorized (invalid token)
- `403` — Forbidden (user doesn't have access)
- `404` — Not found
- `429` — Rate limited
- `500` — Server error

---

## Rate Limiting

- **Per-user rate limit:** 100 requests per minute
- **Endpoint-specific limits:**
  - `/critic/evaluate` — 10 per hour (expensive)
  - `/agents/*/query` — 50 per hour
  - `/health/*` — 100 per day
  
Response includes headers:
```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 87
X-RateLimit-Reset: 1234567890
```

---

## Implementation Notes

### Stack
- **Framework:** FastAPI (async)
- **Auth:** JWT (PyJWT)
- **Database:** PostgreSQL with async driver (asyncpg)
- **Cache:** Redis (aioredis)
- **LLM:** Anthropic Claude API
- **Orchestration:** LangGraph

### Folder structure
```
backend/
├── main.py                    # FastAPI app + router setup
├── config.py                  # Settings from environment
├── auth.py                    # JWT middleware
├── database.py                # PostgreSQL connection pool
├── redis_client.py            # Redis connection
├── memory/
│   ├── manager.py             # MemoryManager class
│   └── models.py              # SQLAlchemy models
├── orchestrator/
│   ├── orchestrator.py        # Orchestrator class (message router)
│   └── state.py               # StateGraph definition
├── agents/
│   ├── base_agent.py          # BaseAgent class
│   ├── life_admin.py
│   ├── wellness.py
│   ├── finance.py
│   ├── content.py
│   ├── relationships.py
│   └── critic.py
├── integrations/
│   ├── notion.py
│   ├── google_calendar.py
│   ├── plaid.py
│   ├── healthkit.py
│   └── feeds.py
├── routers/
│   ├── briefing.py
│   ├── agents.py
│   ├── health.py
│   ├── finance.py
│   ├── content.py
│   ├── contacts.py
│   ├── tasks.py
│   └── memory.py
├── websocket.py               # WebSocket handlers
├── tasks.py                   # Celery/APScheduler jobs
└── logger.py                  # Structured logging
```

