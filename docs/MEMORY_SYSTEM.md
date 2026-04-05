# JARVIS Memory System

## Overview

JARVIS has a **4-tier memory architecture** that enables agents to learn, remember, and make better decisions over time.

```
┌────────────────────────────────────────┐
│  TASK MEMORY                           │
│  (Agent-specific state + streaks)      │
│  TTL: Forever (PostgreSQL)             │
├────────────────────────────────────────┤
│  SEMANTIC MEMORY                       │
│  (Embeddings of insights, decisions)   │
│  TTL: Forever (pgvector + PostgreSQL)  │
├────────────────────────────────────────┤
│  LONG-TERM MEMORY                      │
│  (Historical facts, events, catalog)   │
│  TTL: Forever (PostgreSQL)             │
├────────────────────────────────────────┤
│  SHORT-TERM MEMORY                     │
│  (Current context, today's state)      │
│  TTL: 24 hours (Redis)                 │
└────────────────────────────────────────┘
```

---

## Layer 1: Short-Term Memory (Redis, 24h)

**Purpose:** What's happening *right now*. Current conversation context, today's state.

**Data:**
```redis
briefing:user_123:2026-04-05 → JSON (today's briefing)
health_snapshot:user_123:today → JSON (latest HK sync)
portfolio_value:user_123:today → JSON (current portfolio)
user_mood:user_123:today → JSON (latest mood check-in)
calendar_events:user_123:today → JSON (today's events)
notification_queue:user_123 → LIST (pending notifications)
conversation:session_123 → LIST (current chat messages)
last_refresh:user_123 → TIMESTAMP (when things were last synced)
```

**Who reads it:**
- Frontend (for instant dashboard updates)
- Agents (to know what happened earlier today)
- WebSocket handlers (for real-time updates)

**Who writes it:**
- Agents (when they compute something new)
- HealthKit sync (new HK data arrives)
- Notification manager (when scheduling notifications)
- Time-based triggers (morning, noon, evening)

**TTL:**
- `briefing:*` → 24 hours
- `health_snapshot:*` → 1 hour (refreshes often)
- `portfolio_value:*` → 30 minutes (market hours), 1 hour (after market)
- `notification_queue:*` → until delivered
- `conversation:*` → 24 hours (or until user closes chat)

**Access pattern:**
```python
# Wellness Agent checking if user already logged mood today
mood_today = redis.get(f"user_mood:user_123:today")
if not mood_today:
    # Schedule mood check-in notification
    
# Frontend loading dashboard
briefing = redis.get(f"briefing:user_123:today")
if not briefing:
    # Tell backend to generate one now
```

---

## Layer 2: Long-Term Memory (PostgreSQL)

**Purpose:** The historical record. Every fact, event, interaction, and insight.

**Key tables:**

### user_events (master event log)
```sql
CREATE TABLE user_events (
  id UUID PRIMARY KEY,
  user_id UUID,
  event_type TEXT, -- "workout", "mood_logged", "task_completed", "contact_reached", "trade_executed", etc.
  timestamp TIMESTAMPTZ,
  data JSONB, -- event-specific fields
  agent_source TEXT, -- which agent generated this?
  created_at TIMESTAMPTZ
);
-- Index: (user_id, event_type, timestamp)
```

Example rows:
```json
{ event_type: "workout_completed", data: { type: "strength", duration: 60, calories: 400 } }
{ event_type: "task_completed", data: { task_id: "...", title: "...", category: "startup" } }
{ event_type: "contact_reached", data: { contact_id: "...", channel: "message" } }
{ event_type: "mood_logged", data: { score: 7, note: "good day", energy: 8 } }
{ event_type: "portfolio_updated", data: { value: 5000000, delta: +12500 } }
{ event_type: "bill_paid", data: { bill_id: "...", amount: 2000 } }
```

### user_facts (normalized facts about user)
```sql
CREATE TABLE user_facts (
  id UUID PRIMARY KEY,
  user_id UUID,
  fact_type TEXT, -- "workout_streak", "mood_average_7d", "contact_frequency", etc.
  value JSONB,
  computed_at TIMESTAMPTZ,
  expires_at TIMESTAMPTZ, -- some facts are stale after time
  UNIQUE(user_id, fact_type)
);

Example:
{ fact_type: "workout_streak", value: { days: 8, type: "strength" } }
{ fact_type: "mood_average_7d", value: { average: 6.8, trend: "up" } }
{ fact_type: "contact_urgency", value: { overdue: ["friend_1", "mom"], due_soon: ["dad"] } }
{ fact_type: "portfolio_allocation", value: { equity: 60%, debt: 30%, cash: 10% } }
```

### user_preferences
```sql
CREATE TABLE user_preferences (
  user_id UUID,
  key TEXT, -- "notification_max_per_day", "supplement_targets", "savings_goal", etc.
  value JSONB,
  updated_at TIMESTAMPTZ,
  PRIMARY KEY (user_id, key)
);

Example:
{ key: "notification_max_per_day", value: { normal: 6, intense: 10 } }
{ key: "supplement_targets", value: { creatine_daily: 5, whey_scoops: 3 } }
{ key: "fitness_goals", value: { target_weight: 85, target_body_fat: 12 } }
{ key: "financial_goals", value: { savings_rate_target": 35, emergency_fund_months: 6 } }
```

### Agent-specific tables
Each agent maintains its own facts:

**wellness_snapshots** (one per day)
```sql
CREATE TABLE wellness_snapshots (
  id UUID PRIMARY KEY,
  user_id UUID,
  date DATE,
  sleep_duration_minutes INT,
  sleep_efficiency NUMERIC,
  deep_sleep_minutes INT,
  hrv_ms NUMERIC,
  resting_hr INT,
  steps INT,
  active_calories INT,
  stand_hours INT,
  body_weight_kg NUMERIC,
  recovery_score INT, -- 0-100
  workload_score INT, -- 0-100
  mood INT, -- 1-10
  synced_at TIMESTAMPTZ,
  UNIQUE(user_id, date)
);
```

**holdings** (current portfolio)
```sql
CREATE TABLE holdings (
  id UUID PRIMARY KEY,
  user_id UUID,
  symbol TEXT,
  name TEXT,
  asset_type TEXT, -- equity, mf, etf, gold, fd, crypto, cash
  quantity NUMERIC,
  avg_cost_price NUMERIC,
  current_price NUMERIC,
  current_value NUMERIC,
  unrealized_pnl NUMERIC,
  UNIQUE(user_id, symbol)
);
```

**tasks** (life admin items)
```sql
CREATE TABLE tasks (
  id UUID PRIMARY KEY,
  user_id UUID,
  title TEXT,
  category TEXT, -- work, personal, finance, health, learning, startup
  priority TEXT, -- critical, high, medium, low
  status TEXT, -- not_started, in_progress, done
  due_date DATE,
  completed_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ
);
```

**contacts** (people in user's network)
```sql
CREATE TABLE contacts (
  id UUID PRIMARY KEY,
  user_id UUID,
  name TEXT,
  relationship_type TEXT, -- family, close_friend, friend, professional, acquaintance
  birthday DATE,
  last_contacted DATE,
  contact_frequency TEXT, -- weekly, biweekly, monthly, quarterly
  importance TEXT, -- high, medium, low
  urgency_score NUMERIC, -- computed: (days_since_contact / target_frequency) * importance_weight
  UNIQUE(user_id, name)
);
```

**feed_items** (content agent's curated items)
```sql
CREATE TABLE feed_items (
  id UUID PRIMARY KEY,
  source TEXT, -- techcrunch, hn, arxiv, producthunt, etc.
  url TEXT UNIQUE,
  title TEXT,
  summary TEXT,
  idea_card JSONB, -- rich card if generated
  relevance_score NUMERIC,
  published_at TIMESTAMPTZ,
  tags TEXT[]
);

CREATE TABLE user_feed_interactions (
  id UUID PRIMARY KEY,
  user_id UUID,
  feed_item_id UUID REFERENCES feed_items(id),
  action TEXT, -- viewed, saved, liked, skipped
  interacted_at TIMESTAMPTZ
);
```

---

## Layer 3: Semantic Memory (pgvector)

**Purpose:** Learn from past decisions, recognize similar patterns, inject relevant context.

**How it works:**
1. When agent makes a decision, summarize it
2. Embed summary using Claude's embeddings API
3. Store in PostgreSQL with vector column
4. When agent needs context, semantic search past similar decisions

**Key tables:**

### critic_sessions (with embeddings)
```sql
CREATE TABLE critic_sessions (
  id UUID PRIMARY KEY,
  user_id UUID,
  prompt TEXT, -- what user asked for critique
  category TEXT, -- startup_idea, plan, decision, etc.
  verdict TEXT, -- red, yellow, green
  core_critique TEXT,
  assumptions TEXT,
  missing_info TEXT,
  if_you_must TEXT,
  personal_context TEXT,
  full_response TEXT,
  context_snapshot JSONB, -- recovery, finance, calendar at time
  embedding vector(1536), -- pgvector embedding
  created_at TIMESTAMPTZ
);

CREATE INDEX critic_embedding_idx ON critic_sessions 
  USING ivfflat (embedding vector_cosine_ops) WITH (lists = 100);
```

### agent_insights (general insights from all agents)
```sql
CREATE TABLE agent_insights (
  id UUID PRIMARY KEY,
  user_id UUID,
  agent TEXT, -- which agent generated this?
  insight_type TEXT, -- pattern, warning, opportunity, learning
  text TEXT,
  context JSONB,
  embedding vector(1536),
  created_at TIMESTAMPTZ
);

Example:
{ agent: "wellness", insight_type: "pattern", 
  text: "User works out consistently when mood > 7 but skips when recovery < 50" }
{ agent: "finance", insight_type: "warning",
  text: "User's spending on dining increases 30% during high-stress work periods" }
{ agent: "critic", insight_type: "learning",
  text: "User tends to overestimate time for content creation projects" }
```

**Retrieval:**

When Critic evaluates a startup idea similar to one from 6 months ago:
```python
# Critic embeds: "Should I build a personal AI assistant for founders?"
current_embedding = embeddings.embed("startup_idea", "personal_ai_assistant")

# Semantic search for similar past critiques
similar_critiques = db.query("""
  SELECT prompt, verdict, core_critique, reasoning
  FROM critic_sessions
  WHERE user_id = ?
  ORDER BY (embedding <-> ?) LIMIT 5
""", [user_id, current_embedding])

# Inject as context: "You've evaluated similar ideas before..."
```

**Use cases:**
- "Have I evaluated this type of idea before?"
- "What patterns does the system know about me?"
- "What context from 3 months ago is still relevant?"
- "Am I repeating the same mistake?"

---

## Layer 4: Task Memory (Agent-Specific State)

**Purpose:** Track agent-specific, derived state (streaks, patterns, thresholds).

**Examples:**

### Wellness Agent's task memory
```python
{
  "workout_streak": {
    "days": 8,
    "type": "strength",
    "last_date": "2026-04-04"
  },
  "mood_trend_7d": {
    "average": 6.8,
    "lowest": 5,
    "highest": 8,
    "trend": "improving"
  },
  "supplement_streak": {
    "creatine": 24,  # days
    "whey": 18
  },
  "burnout_risk": {
    "recovery_score_trend": "declining",
    "consecutive_low_sleep_nights": 3,
    "workload_trend": "increasing"
  }
}
```

### Finance Agent's task memory
```python
{
  "portfolio_allocation": {
    "equity_pct": 62,
    "debt_pct": 25,
    "cash_pct": 13
  },
  "sector_exposure": {
    "tech": 45,
    "healthcare": 20,
    "finance": 15,
    "other": 20
  },
  "emergency_fund_coverage": {
    "months": 5.2,
    "trend": "stable"
  },
  "savings_rate_momentum": {
    "this_month": 38,
    "last_month": 35,
    "trend": "up"
  }
}
```

### Relationship Agent's task memory
```python
{
  "contact_urgency_scores": {
    "contact_1": 0.8,  # high (overdue by 20 days)
    "contact_2": 0.3,  # low (contacted 2 days ago)
  },
  "reach_out_streaks": {
    "contact_1": 0,  # broken (missed last check-in)
    "contact_2": 4  # 4 check-ins in a row
  },
  "upcoming_birthdays": [
    { contact_id: "...", days_until: 3 }
  ]
}
```

**Where stored:**
- PostgreSQL `agent_task_state` table
- Refreshed periodically (e.g., every time agent runs)
- Available for both current and historical analysis

---

## Memory Write Operations

Every meaningful action triggers a memory write:

### Event broadcast → writes to memory
```python
await orchestrator.broadcast_event(
    agent="wellness",
    event_type="workout_completed",
    payload={...}
)
# Automatically writes:
# 1. → user_events table (for history)
# 2. → Redis cache (for today's view)
# 3. → wellness_snapshots (aggregated daily)
```

### Agent query answered → writes to memory
```python
response = await orchestrator.query(
    to_agent="wellness",
    query="recovery_score_today"
)
# Automatically logs:
# → agent_communications table (for audit trail)
```

### Critique session → writes with embedding
```python
critique = await orchestrator.command_agent(
    to_agent="critic",
    command="evaluate",
    params={...}
)
# Automatically:
# 1. Stores critique in critic_sessions
# 2. Embeds full_response
# 3. Stores in pgvector
# 4. Available for future semantic search
```

---

## Memory Query Patterns

### Pattern 1: Fetch current state (Redis)
```python
# Frontend: "Show me today's briefing"
briefing = redis.get(f"briefing:user_{user_id}:today")
if not briefing:
    trigger_briefing_generation()
```

### Pattern 2: Fetch historical facts (PostgreSQL)
```python
# Wellness Agent: "What's user's 7-day average mood?"
average_mood = db.query("""
  SELECT AVG(score) FROM mood_logs
  WHERE user_id = ? AND logged_at > NOW() - INTERVAL '7 days'
""", [user_id])
```

### Pattern 3: Semantic search (pgvector)
```python
# Critic: "Have I evaluated similar ideas?"
embedding = embeddings.embed(user_prompt)
similar = db.query("""
  SELECT prompt, verdict FROM critic_sessions
  WHERE user_id = ? 
  ORDER BY (embedding <-> ?) LIMIT 5
""", [user_id, embedding])
```

### Pattern 4: Compute derived facts (PostgreSQL + cache)
```python
# Relationship Agent: "Who should I contact today?"
contacts = db.query("""
  SELECT id, name, urgency_score
  FROM contacts
  WHERE user_id = ? 
  ORDER BY urgency_score DESC
  LIMIT 3
""", [user_id])
# Cache result in Redis for 1 hour
redis.setex(f"reach_outs:{user_id}", 3600, contacts)
```

---

## Memory Retention Policy

| Data Type | Retention | Reason |
|---|---|---|
| **Short-term (Redis)** | 24 hours | Just today's context |
| **Events** | 5 years | Legal requirement, historical analysis |
| **Health snapshots** | Forever | Detect long-term patterns |
| **Transactions** | Forever | Tax records, trend analysis |
| **Contacts** | Forever | Relationship history |
| **Critiques** | Forever | Learn from past decisions |
| **Preferences** | Forever | User settings |
| **Logs** | 90 days | Debugging, not long-term value |

---

## Memory Privacy & Access

**What agents can read:**
- All agents can read facts relevant to their domain
- Wellness agent doesn't read Finance data (unless asked by another agent)
- Content agent doesn't read Health data (privacy)

**What agents can write:**
- Agents write only their own domain facts
- Central Orchestrator writes cross-domain summaries (for briefing)

**User access:**
- User can export all memory in JSON/CSV
- User can delete memory (right to be forgotten)
- User can view audit trail of what agents know

---

## Implementation: Building the Memory System

### Step 1: Set up PostgreSQL tables
```sql
-- Master event log
CREATE TABLE user_events (...);

-- Facts
CREATE TABLE user_facts (...);

-- Time-series
CREATE TABLE wellness_snapshots (...);
CREATE TABLE holdings (...);
CREATE TABLE mood_logs (...);

-- pgvector for semantic search
CREATE EXTENSION IF NOT EXISTS vector;
CREATE TABLE critic_sessions (
  ...,
  embedding vector(1536)
);
CREATE INDEX ON critic_sessions USING ivfflat (embedding);

-- Audit trail
CREATE TABLE agent_communications (...);
CREATE TABLE agent_task_state (...);
```

### Step 2: Set up Redis for caching
```python
import redis

r = redis.Redis(host='localhost', port=6379, db=0)

# Caching patterns
def cache_briefing(user_id, briefing_data):
    r.setex(f"briefing:{user_id}:{date.today()}", 86400, json.dumps(briefing_data))

def get_cached_briefing(user_id):
    return r.get(f"briefing:{user_id}:{date.today()}")
```

### Step 3: Create Memory Manager class
```python
class MemoryManager:
    def __init__(self, db, redis, embeddings_client):
        self.db = db
        self.cache = redis
        self.embeddings = embeddings_client
    
    async def write_event(self, user_id, event_type, data, agent_source):
        # Write to user_events
        # Write to cache if relevant
        # Update derived facts if needed
    
    async def query_facts(self, user_id, fact_type):
        # Query from PostgreSQL
        # Cache if frequently accessed
    
    async def semantic_search(self, user_id, embedding, limit=5):
        # pgvector similarity search
        # Return relevant past insights
    
    async def update_task_state(self, agent, task_state):
        # Write agent-specific state
```

### Step 4: Integrate with agents
```python
class WellnessAgent:
    def __init__(self, orchestrator, memory):
        self.orchestrator = orchestrator
        self.memory = memory
    
    async def compute_recovery_score(self, user_id):
        # Read from memory
        sleep_data = await self.memory.query_facts(user_id, "sleep_last_night")
        hrv_data = await self.memory.query_facts(user_id, "hrv_latest")
        mood_data = await self.memory.query_facts(user_id, "mood_today")
        
        # Compute
        recovery_score = compute_score(sleep_data, hrv_data, mood_data)
        
        # Write back
        await self.memory.write_event(
            user_id, "recovery_score_computed",
            { "score": recovery_score, "factors": {...} },
            agent_source="wellness"
        )
        
        return recovery_score
```

---

## Monitoring & Debugging

### Memory health checks
```python
async def check_memory_health(user_id):
    # Is today's briefing cached?
    # Are facts up to date?
    # Is pgvector index healthy?
    # Any missing data?
    
    return {
        "cache_fresh": True,
        "facts_stale": False,
        "vector_index_size": 1523,
        "memory_size_gb": 2.3
    }
```

### Memory analytics
```python
# How many events this month?
# Which facts are most frequently queried?
# Which agent communications are most valuable?
# Average time from event to memory write?
```

