# JARVIS Quick Reference Guide

## One-Page Overview

JARVIS is a **personal multi-agent AI OS** where:

1. **6 specialized agents** work independently but coordinate through an **Orchestrator**
2. **Agents talk to each other** via the Orchestrator (query, event, command patterns)
3. **Rich 4-tier memory** lets agents learn and context-aware
4. **Backend does all thinking**, frontend just displays data
5. **Users interact** via dashboard, critic sessions, or mobile notifications

---

## Architecture in 30 Seconds

```
┌─────────────────────┐
│   Frontend (Web)    │  Just shows data
│   REST API calls    │  Collects user input
└──────────┬──────────┘
           │
    ┌──────▼──────┐
    │  Backend    │  All business logic
    │  FastAPI    │  All AI reasoning
    └──────┬──────┘
           │
    ┌──────▼──────────────────┐
    │  Orchestrator LangGraph  │  Routes messages
    │  ┌─────────────────────┐ │  Manages state
    │  │ Agent-to-agent      │ │
    │  │ communication layer │ │
    │  └─────────────────────┘ │
    └──────┬──────────────────┘
           │
    ┌──────▼──────┬──────────┬──────────┐
    │  6 Agents   │  Memory  │  Integ.  │
    │             │  System  │  APIs    │
    └─────────────┴──────────┴──────────┘
```

---

## 6 Agents at a Glance

| Agent | Purpose | Inputs | Outputs |
|---|---|---|---|
| **LifeAdmin** | Eliminate open loops | Tasks, Calendar, Bills | Daily inbox, Alerts, Prep notes |
| **Wellness** | Keep you performing | HealthKit, Mood, Calendar | Recovery score, Workout rec, Warnings |
| **Finance** | Complete awareness | Plaid/Smallcase, Prices, Transactions | Portfolio delta, Cashflow, Advice |
| **Content** | Curated daily feed | RSS, HN, arXiv, ProductHunt | Daily feed, Idea cards, Facts |
| **Relationships** | Prevent FOMO | Notion contacts, Interaction history | Reach-outs, Birthdays, Messages |
| **Critic** | Brutally honest feedback | User prompt + context from other agents | Verdict + reasoning, Assumptions, Action items |

---

## Communication Patterns

### Agent Queries Another Agent
```python
# Inside Critic Agent
recovery = await orchestrator.query(
    from_agent="critic",
    to_agent="wellness",
    query="is_user_healthy_for_stress",
    context={"stress_level": "high"}
)
# Wellness responds synchronously
```

### Agent Broadcasts Event
```python
# Inside Wellness Agent
await orchestrator.broadcast_event(
    agent="wellness",
    event_type="workout_completed",
    payload={"type": "strength", "duration": 60}
)
# All agents can react; memory records it
```

### Orchestrator Commands Agent
```python
# Inside daily briefing workflow
briefing = await orchestrator.command_agent(
    to_agent="life_admin",
    command="get_daily_inbox",
    params={"date": "2026-04-05"}
)
```

---

## Memory System (4 Layers)

| Layer | Storage | TTL | Use |
|---|---|---|---|
| **Short-term** | Redis | 24h | Current context, today's state |
| **Long-term** | PostgreSQL | Forever | Historical events, facts, catalog |
| **Semantic** | pgvector | Forever | Embeddings for similarity search |
| **Task-specific** | PostgreSQL | Forever | Agent-specific derived state |

**Example query:**
```python
# Fetch today's recovery score (short-term)
recovery = redis.get("recovery_score:user_123:today")

# Fetch 7-day sleep trend (long-term)
sleep_data = db.query("""
  SELECT date, duration FROM wellness_snapshots
  WHERE user_id = ? AND date > NOW() - 7 DAYS
""")

# Find similar past critiques (semantic)
embeddings = db.query("""
  SELECT prompt, verdict FROM critic_sessions
  WHERE user_id = ? ORDER BY (embedding <-> ?) LIMIT 5
""")
```

---

## Backend API Summary

**Auth:** All requests need JWT Bearer token

**Endpoints:**
- `GET /briefing/today` — Daily briefing (all agents merged)
- `POST /agents/critic/evaluate` — Ask Critic (with context)
- `GET /agents/{agent}/query` — Ask specific agent
- `GET /health/snapshots/latest` — Latest health data
- `GET /finance/holdings` — Portfolio holdings
- `GET /content/feed/today` — Curated feed
- `GET /contacts/reach-outs-today` — People to contact
- `GET /tasks/inbox-today` — Today's tasks
- `WS /ws/updates` — Real-time updates

---

## Frontend (Thin Layer)

**Tech:** React (web) or SwiftUI (iOS)

**Responsibilities:**
- ✅ Display API responses
- ✅ Collect user input
- ✅ Cache locally (offline)
- ✅ Handle authentication

**What NOT to do:**
- ❌ LLM calls (backend does)
- ❌ Agent orchestration (backend does)
- ❌ Business logic (backend does)
- ❌ Data persistence (backend does)

**Key screens:**
1. **Home** — Dashboard with all briefing data
2. **Wellness** — Sleep, workouts, recovery, mood
3. **Finance** — Holdings, net worth, cashflow
4. **Content** — Daily curated feed, idea cards
5. **Relationships** — People CRM, reach-outs, birthdays
6. **Critic** — Text/voice input, past sessions, results

---

## Implementation Phases (10 Weeks)

| Week | What | Done When |
|---|---|---|
| 1 | Backend setup, Orchestrator, Auth, Base agents | Can receive requests |
| 2 | Connect to DB, fetch real data, HealthKit sync | Agents read from memory |
| 3 | Inter-agent communication, LLM integration | Critic can query others |
| 4 | Daily briefing workflow, scheduling | All agents work together |
| 5 | Frontend home screen, tab navigation | Can see data |
| 6 | Memory + semantic search | Learn from past |
| 7 | Polish, error handling, logging | Production-ready |
| 8 | Deploy backend + frontend | Live on internet |
| 9 | iOS app (optional) | Native features |
| 10 | App Store launch | Public release |

---

## How Data Flows

### Morning (6:45 AM, Briefing Generated)

```
1. Cron fires: generate_daily_briefing(user_id)
   ↓
2. Orchestrator calls all agents in parallel:
   - LifeAdmin: "Get today's tasks"
   - Wellness: "Get recovery score"
   - Finance: "Get portfolio delta"
   - Content: "Get top stories"
   - Relationships: "Get reach-outs"
   ↓
3. All responses merge → DailyBriefing object
   ↓
4. Memory writes briefing to:
   - Redis (for instant frontend access)
   - PostgreSQL (for history)
   ↓
5. Frontend fetches GET /briefing/today
   ↓
6. User sees dashboard with all data
```

### User Asks Critic (Anytime)

```
1. User submits: "Should I launch a course?"
   ↓
2. Frontend: POST /agents/critic/evaluate
   ↓
3. Critic Agent runs:
   a. Ask Wellness: "User's recovery score?"
   b. Ask Finance: "Can user afford 3 months unpaid?"
   c. Ask LifeAdmin: "Calendar load?"
   d. Fetch past similar critiques (semantic search)
   ↓
4. Critic calls Claude with all context
   ↓
5. Claude returns structured critique:
   - Verdict (red/yellow/green)
   - Core critique
   - Assumptions
   - Missing info
   - If you must proceed
   - Personal context
   ↓
6. Memory saves critique (with embedding for future)
   ↓
7. Frontend displays result
```

### User Completes Workout

```
1. HealthKit detects workout
   ↓
2. iOS app calls: POST /health/sync { workout_data }
   ↓
3. Backend broadcasts: event_type="workout_completed"
   ↓
4. Memory writes event to:
   - user_events table
   - Redis cache
   - wellness_snapshots (aggregated)
   ↓
5. All agents can react:
   - Wellness: update recovery score
   - LifeAdmin: unlock time (user not available)
   - Critic: note for future "can user handle pressure"
   ↓
6. Frontend refreshes (via WebSocket or next fetch)
   ↓
7. Maybe notification: "Log your whey protein?"
```

---

## Database Schema (Core Tables)

```sql
-- Users and preferences
users (id, email, name, created_at)
user_preferences (user_id, key, value)

-- Master event log
user_events (id, user_id, event_type, data, agent_source, created_at)

-- Time series (one row per day per user)
wellness_snapshots (user_id, date, sleep, recovery, hrv, mood, ...)
holdings (user_id, symbol, quantity, price, pnl, ...)
tasks (user_id, title, category, priority, status, due_date, ...)
contacts (user_id, name, relationship, birthday, urgency, ...)
mood_logs (user_id, timestamp, score, energy, note)
feed_items (source, url, title, summary, tags, relevance_score)
user_feed_interactions (user_id, feed_item_id, action, timestamp)

-- Semantic memory
critic_sessions (user_id, prompt, verdict, critique, embedding, created_at)

-- Audit trail
agent_communications (user_id, from_agent, to_agent, message_type, payload, created_at)
```

---

## Key Integrations

| Integration | Purpose | Implementation |
|---|---|---|
| **Google Calendar** | Get events for next 7 days | OAuth 2.0 → Google Calendar API |
| **Notion** | Read/write tasks, bills, contacts, knowledge cards | OAuth 2.0 → Notion API |
| **HealthKit** (iOS) | Sync workouts, sleep, HRV, heart rate | iOS app → POST /health/sync |
| **Plaid / Smallcase** | Get portfolio holdings & transactions | OAuth 2.0 → Plaid/Smallcase API |
| **News APIs** | Fetch AI/tech/startup news | REST calls to RSS, HN, arXiv, ProductHunt, NewsAPI |
| **Anthropic Claude** | LLM for all AI reasoning | API calls with prompts |

---

## Environment Variables Needed

```bash
# Database
DATABASE_URL=postgresql://user:pass@host/jarvis
REDIS_URL=redis://host:6379

# Auth
JWT_SECRET_KEY=your-secret-key

# LLM
ANTHROPIC_API_KEY=sk-...

# Integrations
GOOGLE_CLIENT_ID=...
GOOGLE_CLIENT_SECRET=...
NOTION_CLIENT_ID=...
NOTION_CLIENT_SECRET=...
PLAID_CLIENT_ID=...
PLAID_SECRET=...

# APNs (if using iOS)
APNS_TEAM_ID=...
APNS_KEY_ID=...
APNS_PRIVATE_KEY=...
APNS_BUNDLE_ID=com.yourname.jarvis

# Frontend
API_BASE_URL=https://api.jarvis.yourdomain.com/v1
```

---

## Running Locally

```bash
# Prerequisites
# - Python 3.11+
# - Node.js 18+
# - PostgreSQL 14+
# - Redis

# Backend
cd backend
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
cp .env.example .env  # Fill in secrets
python main.py
# Server at http://localhost:8000
# Docs at http://localhost:8000/docs

# Frontend
cd frontend
npm install
npm start
# App at http://localhost:3000

# Database
psql -U postgres -d jarvis
\dt  # List tables
SELECT * FROM user_events LIMIT 10;  # Inspect data
```

---

## Deployment Checklist

### Backend
- [ ] Code pushed to GitHub
- [ ] Tests passing
- [ ] Environment variables set in production
- [ ] Database migrated
- [ ] Static file serving configured
- [ ] CORS configured for frontend domain
- [ ] Logging/monitoring set up (Sentry, DataDog, etc.)
- [ ] Deployed to AWS ECS / Heroku / Railway / etc.
- [ ] Domain/SSL certificate configured

### Frontend
- [ ] Build passes
- [ ] API_BASE_URL points to production backend
- [ ] Auth tokens stored securely
- [ ] Offline caching works
- [ ] Deployed to Vercel / Netlify / etc.
- [ ] Domain configured

### iOS (Optional)
- [ ] Bundle ID matches APNS config
- [ ] TestFlight build created
- [ ] Privacy policy written
- [ ] App Store submission prepared

---

## Testing Strategy

**Backend:**
```python
# Unit tests for each agent
def test_wellness_recovery_score():
    agent = WellnessAgent(mock_orchestrator, mock_memory)
    response = await agent.handle_query("recovery_score_today", {})
    assert response["recovery_score"] >= 0
    assert response["recovery_score"] <= 100

# Integration tests for workflows
async def test_daily_briefing_workflow():
    briefing = await generate_daily_briefing(orchestrator, "user_123")
    assert "life_admin" in briefing
    assert "wellness" in briefing
    assert briefing["date"] == str(date.today())
```

**Frontend:**
```javascript
// Component tests
test("Home screen displays recovery score", () => {
  const briefing = { wellness: { recovery_score: 62 } };
  render(<Home briefing={briefing} />);
  expect(screen.getByText(/62/)).toBeInTheDocument();
});

// API tests
test("fetchBriefing calls correct endpoint", async () => {
  await fetchBriefing();
  expect(fetch).toHaveBeenCalledWith("/briefing/today");
});
```

---

## Debugging Tips

**Agent not responding:**
```python
# Check orchestrator logs
print(orchestrator.messages)  # See all messages

# Check agent registered
print(orchestrator.agents.keys())  # Should include agent name

# Check agent method exists
agent = orchestrator.agents["wellness"]
print(dir(agent))  # Should include handle_query, handle_event, etc.
```

**Data not in memory:**
```sql
-- Check if event was written
SELECT * FROM user_events WHERE event_type = 'workout_completed' LIMIT 1;

-- Check Redis cache
redis-cli
GET "recovery_score:user_123:today"

-- Check aging/deletion
SELECT * FROM wellness_snapshots WHERE date = '2026-04-05';
```

**Frontend not updating:**
```javascript
// Check network calls
window.fetch // See all requests in DevTools Network tab

// Check localStorage
localStorage.getItem("briefing")

// Check console for errors
console.log(error.message)
```

---

## FAQ

**Q: How does the system handle user privacy?**
A: No secrets hardcoded. HealthKit data encrypted in transit. Finance credentials stored server-side only. User can export/delete all data.

**Q: What if an agent crashes?**
A: Orchestrator catches errors and retries. Failed events logged. User notified via error message. System keeps running.

**Q: How do I add a new agent?**
A: 
1. Extend BaseAgent
2. Implement handle_query, handle_event, handle_command
3. Register with orchestrator: `orchestrator.register_agent("new_agent", NewAgent(...))`
4. Add endpoint in routers/

**Q: Can agents run in parallel?**
A: Yes. Orchestrator supports async/await. Use `asyncio.gather()` for parallel queries.

**Q: What's the latency?**
A: 
- Daily briefing: 2-3 seconds (parallel agent calls)
- Critic evaluation: 5-10 seconds (LLM call)
- Query a single agent: 200-500ms (DB fetch)
- WebSocket update: <100ms

**Q: How do I scale?**
A: 
- Database: Use RDS Multi-AZ
- Cache: Use ElastiCache
- Backend: Use load balancer + multiple ECS instances
- Frontend: Use CDN (CloudFront)
- Queue: Use SQS for long-running jobs

---

## Resources

- **FastAPI:** https://fastapi.tiangolo.com/
- **LangGraph:** https://langchain-ai.github.io/langgraph/
- **SQLAlchemy:** https://www.sqlalchemy.org/
- **React:** https://react.dev/
- **PostgreSQL:** https://www.postgresql.org/docs/
- **Anthropic Claude:** https://docs.anthropic.com/

