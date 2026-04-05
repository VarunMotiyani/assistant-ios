# Agent Communication Protocol

## Overview

Agents communicate through the **Orchestrator** using a **message-based architecture**. Agents never call each other directly — all communication is routed through the Orchestrator, which maintains a shared conversation state and memory log.

---

## Message Types

### 1. **Query Messages** (Agent → Orchestrator → Other Agents)

Agent asks a question and waits for a response.

```python
# Critic Agent wants to know if user is healthy enough for startup work
response = await orchestrator.query(
    from_agent="critic",
    to_agent="wellness",
    query="is_user_healthy_for_stress",
    context={
        "stress_level": "high",
        "duration_months": 3
    }
)
# Response includes reasoning from Wellness agent
```

**Use cases:**
- Critic querying Wellness, Finance, LifeAdmin for context
- Finance querying Wellness for stress-related spending patterns
- Any agent needing live data from another

### 2. **Event Messages** (Agent → Orchestrator → All Agents + Memory)

Agent broadcasts an event. All agents can react.

```python
# Wellness Agent detected a workout
await orchestrator.broadcast_event(
    agent="wellness",
    event_type="workout_completed",
    payload={
        "workout_id": "uuid",
        "type": "strength",
        "duration_minutes": 60,
        "calories_burned": 400,
        "avg_heart_rate": 145,
        "timestamp": "2026-04-05T06:30:00Z"
    }
)

# Orchestrator:
# 1. Writes to memory (health_snapshots table)
# 2. Notifies all agents subscribed to "workout_completed"
# 3. May trigger reactive behaviors (e.g., "suggest whey protein post-workout")
```

**Use cases:**
- User completed a task → broadcast to LifeAdmin, Critic, Finance (freed time)
- User logged mood → broadcast to Wellness (pattern detection)
- Market news item saved → broadcast to Finance (relevant for portfolio)
- Contact reached out → broadcast to Relationship (interaction logged)

### 3. **Command Messages** (Orchestrator → Agent → Action)

Orchestrator tells an agent to do something.

```python
# Daily briefing workflow tells LifeAdmin to get today's tasks
await orchestrator.command_agent(
    to_agent="life_admin",
    command="get_daily_inbox",
    params={
        "date": "2026-04-05",
        "include_critical_alerts": True
    }
)
# LifeAdmin executes, returns structured data
```

**Use cases:**
- Workflows (daily briefing, weekly review, etc.)
- User-triggered actions (open "wellness" panel → load wellness data)
- Time-triggered jobs (market open alert)

### 4. **Proposal Messages** (Agent → Orchestrator → Decision)

Agent proposes an action. Orchestrator decides (or asks other agents).

```python
# Wellness Agent proposes: "User should rest today"
proposal = await orchestrator.propose_action(
    from_agent="wellness",
    action="suggest_rest_day",
    reasoning="Recovery score 35, sleep 5.5h, workload 80",
    priority="high"
)

# Orchestrator may:
# 1. Accept immediately (no conflicts)
# 2. Query LifeAdmin: "Does user have critical tasks?"
# 3. If conflict, escalate to Critic for tiebreaker
# 4. Return decision + reasoning to frontend
```

**Use cases:**
- Wellness proposes rest day (conflicts with LifeAdmin's tasks)
- Finance proposes sell signal (conflicts with user's beliefs)
- Critic proposes "don't do this" (overrides other agents)

---

## Message Flow (Detailed)

### Query Message Flow

```
┌─────────────────┐
│ Critic Agent    │
│ (needs context) │
└────────┬────────┘
         │
         │ await orchestrator.query(
         │    to_agent="wellness",
         │    query="recovery_score_today"
         │ )
         │
         ▼
┌─────────────────────────────────────┐
│ Orchestrator Message Router          │
│ 1. Log message to memory             │
│ 2. Find Wellness Agent               │
│ 3. Pass to Wellness Agent            │
│ 4. Wait for response                 │
│ 5. Log response to memory            │
│ 6. Return to Critic                  │
└────────┬────────────────────────────┘
         │
         ▼
┌──────────────────────┐
│ Wellness Agent       │
│ 1. Receive query     │
│ 2. Fetch from memory │
│ 3. Compute score     │
│ 4. Return response   │
└──────────────────────┘
         │
         │ response = {
         │   recovery_score: 62,
         │   sleep: 7.2h,
         │   hrv: 45ms,
         │   mood: 7,
         │   reasoning: "..."
         │ }
         │
         ▼
┌─────────────────┐
│ Critic Agent    │
│ Uses response   │
└─────────────────┘
```

### Event Message Flow

```
┌──────────────────┐
│ Wellness Agent   │
│ (HK sync fired)  │
└────────┬─────────┘
         │
         │ await orchestrator.broadcast_event(
         │    event_type="workout_completed",
         │    payload={...}
         │ )
         │
         ▼
┌──────────────────────────────────────┐
│ Orchestrator Event Router            │
│ 1. Write event to memory             │
│ 2. Find all subscribed agents        │
│ 3. Notify each agent                 │
│ 4. Agents can react (optional)       │
└──────────────────────────────────────┘
         │
         ├─────────────────┬──────────────────┬────────────────┐
         │                 │                  │                │
         ▼                 ▼                  ▼                ▼
    ┌─────────┐      ┌─────────┐      ┌──────────┐      ┌─────────┐
    │ LifeAdmin│      │ Finance │      │ Critic   │      │Content  │
    │ (ignores)│      │(ignores)│      │(ignores) │      │(ignores)│
    └─────────┘      └─────────┘      └──────────┘      └─────────┘

    # But Memory is updated
    # health_snapshots table gets new row
```

---

## Agent Responsibilities & Inputs/Outputs

### LifeAdmin Agent

**Inputs:**
- Notion Tasks DB (name, due date, status, priority)
- Notion Bills DB (amount, due date, recurrence)
- Google Calendar (events for next 7 days)
- PostgreSQL tasks table (local cache)

**Outputs:**
- `DailyInbox` — top 3 tasks due today, sorted by importance
- `CriticalAlerts` — bills due in 2 days, overdue items
- `UpcomingWarnings` — events next 3-7 days that need prep
- `CalendarPrepNotes` — for each meeting tomorrow, suggested prep

**Queries it receives:**
- "What's the most critical task today?"
- "Does user have time for X hours of work?"
- "What bills are due this week?"

**Events it broadcasts:**
- `task_created` — user added new task
- `task_completed` — user marked task done
- `task_rescheduled` — user moved due date
- `bill_marked_paid` — user paid a bill

**Query it makes of others:**
- Asks Wellness: "Is user healthy enough for heavy work day?"
- Asks Finance: "Should user prioritize earning tasks?"

---

### Wellness Agent

**Inputs:**
- HealthKit sync (sleep, workouts, heart rate, steps)
- PostgreSQL health_snapshots (latest data)
- User mood check-ins (1-10 scale, optional note)
- Supplement logs (creatine, whey protein)
- Calendar (meeting count per day)

**Outputs:**
- `RecoveryScore` — 0-100 composite score
- `WorkloadScore` — 0-100 (meeting hours + task volume)
- `CoachingNote` — 2-3 sentences personalized to user
- `WorkoutRecommendation` — rest / light / moderate / heavy
- `BurnoutWarning` — if recovery < 40 and workload > 70

**Queries it receives:**
- "Is user recovered enough for startup work?"
- "What's today's recovery score?"
- "Should user exercise today?"

**Events it broadcasts:**
- `health_snapshot_updated` — new HK data synced
- `workout_completed` — user finished workout
- `mood_logged` — user checked in mood
- `sleep_data_synced` — sleep metrics updated
- `burnout_detected` — recovery dangerous

**Query it makes of others:**
- Asks LifeAdmin: "How many meetings today?"
- Asks Critic: "Is user stressed about something?"

---

### Finance Agent

**Inputs:**
- Plaid / Smallcase / Kite API (portfolio holdings, transactions)
- PostgreSQL holdings table (latest prices, P&L)
- PostgreSQL transactions (income, expenses by category)
- Market indices (Nifty50, Sensex, Nasdaq, S&P500)

**Outputs:**
- `PortfolioDelta` — today's gain/loss (INR + %)
- `NetWorthSnapshot` — total portfolio + cash + savings
- `CashflowSummary` — monthly income vs expense
- `SavingsRate` — monthly savings as % of income
- `FinancialAdvice` — LLM-generated insights (3 specific, harsh recommendations)
- `MarketPulse` — index movements, portfolio correlation

**Queries it receives:**
- "What's the portfolio delta today?"
- "Can user afford to take 3 months unpaid?"
- "What's the emergency fund coverage?"

**Events it broadcasts:**
- `portfolio_updated` — prices refreshed
- `transaction_recorded` — new transaction
- `portfolio_alert` — portfolio dropped > 2%
- `cashflow_computed` — monthly summary ready

**Query it makes of others:**
- Asks Wellness: "Is user stressed? (spending patterns)"
- Asks LifeAdmin: "Is there urgent expense coming?"

---

### Content Agent

**Inputs:**
- RSS feeds (TechCrunch, Verge, Ars Technica, etc.)
- Hacker News API (top 30 stories daily)
- arXiv API (AI/ML papers, last 24h)
- ProductHunt API (top 5 launches)
- NewsAPI.org (AI + startup keywords)
- User engagement history (saves, skips, interactions)

**Outputs:**
- `DailyFeed` — 8-12 curated items ranked by relevance
- `TopStories` — top 3 for home screen
- `IdeaCards` — for 1-2 items, rich summaries with talking points
- `KnowledgeFacts` — 1 random interesting fact per day
- Weekly digest

**Queries it receives:**
- "What are today's top AI stories?"
- "Any news about LLM deployment?"

**Events it broadcasts:**
- `feed_refreshed` — new items available
- `idea_card_generated` — rich card ready
- `user_saved_article` — user saved something

**Query it makes of others:**
- Asks Critic: "Is this trend relevant to user's ideas?"

---

### Relationship Agent

**Inputs:**
- Notion People DB (name, birthday, relationship type, frequency, importance)
- PostgreSQL contacts table (local cache)
- Contact interaction history (calls, messages, emails)
- Calendar (detect when contact mentioned in events)

**Outputs:**
- `TodayReachOuts` — 2-3 people to contact today
- `UpcomingMoments` — birthdays/anniversaries next 14 days
- `MessageDrafts` — suggested opening for each person
- RelationshipHealthScore per contact

**Queries it receives:**
- "Who should I reach out to today?"
- "When's [name]'s birthday?"
- "Suggest a message for [contact]"

**Events it broadcasts:**
- `contact_created` — new person added
- `contact_reached_out` — user contacted someone
- `birthday_approaching` — upcoming birthday
- `contact_overdue` — haven't talked in 2x frequency

---

### Critic Agent

**Inputs:**
- User prompt (explicit text or voice)
- Context auto-injected:
  - Current calendar load (LifeAdmin)
  - Recovery score (Wellness)
  - Net worth, savings rate (Finance)
  - Recent saved articles (Content)
- Past critic sessions (pgvector semantic search)
- User-defined goals

**Outputs:**
- `Verdict` — red / yellow / green with one-liner
- `CoreCritique` — 3-5 specific problems
- `Assumptions` — what's risky
- `MissingInfo` — what to research
- `IfYouMustProceed` — 3 changes to improve
- `PersonalContext` — how it fits user's life right now

**Queries it receives:**
- (Direct from user via API, not from agents)
- May query other agents during evaluation

**Events it broadcasts:**
- `critique_session_created` — new session
- `verdict_reached` — critique complete

**Query it makes of others:**
- Asks Wellness: "Is user healthy enough for this?"
- Asks Finance: "Can user afford this?"
- Asks LifeAdmin: "Does user have bandwidth?"

---

## Conflict Resolution

When two agents propose conflicting actions:

```
Scenario:
- Wellness Agent: "User should rest today (recovery 35)"
- LifeAdmin Agent: "User has 3 critical tasks due today"

Resolution:
1. Orchestrator detects conflict
2. Orchestrator asks Critic: "User is proposing: rest vs. work tasks. What's the call?"
3. Critic evaluates:
   - Rest long-term health: critical
   - Short-term task impact: medium
   - User's runways: 8 months savings
   - Verdict: REST. Reschedule tasks.
4. Orchestrator sends to LifeAdmin: "Deprioritize today, reschedule non-critical"
5. User gets notification: "Rest recommended. [N] tasks rescheduled."
```

---

## Memory Write Triggers

Agents trigger memory writes on:

1. **Events** — `broadcast_event()` auto-writes
2. **Queries answered** — query response logged
3. **Proposals made** — proposals logged with decision
4. **Critiques delivered** — full critique with context snapshot
5. **Time-based** — periodic snapshots (daily health, weekly finance)

Example memory entry:
```json
{
  "event_type": "agent_query",
  "timestamp": "2026-04-05T08:15:00Z",
  "from_agent": "critic",
  "to_agent": "wellness",
  "query": "is_user_healthy_for_stress",
  "response": {
    "healthy": false,
    "recovery_score": 35,
    "reasoning": "Poor sleep 5 nights, high workload"
  },
  "memory_id": "uuid"
}
```

This gets:
- Stored in PostgreSQL (agent_communications table)
- Available for future semantic search
- Visible in debug logs
- Exportable in user's memory export

---

## Implementation: How to Build This

### Backend (Python)

1. **Orchestrator class** (orchestrator.py)
   ```python
   class Orchestrator:
       async def query(self, to_agent, query, context):
           # Log to memory
           # Route to agent
           # Wait for response
           # Log response
           # Return
       
       async def broadcast_event(self, agent, event_type, payload):
           # Log to memory
           # Notify all subscribed agents
           # Update any time-series tables
   
       async def command_agent(self, to_agent, command, params):
           # Direct command execution
       
       async def propose_action(self, from_agent, action, reasoning, priority):
           # Evaluate conflicts
           # Query other agents if needed
           # Return decision
   ```

2. **Each Agent class** (agents/wellness.py, etc.)
   ```python
   class WellnessAgent:
       def __init__(self, orchestrator):
           self.orchestrator = orchestrator
       
       async def handle_query(self, query_type, context):
           # Process query
           # Return response
       
       async def handle_event(self, event):
           # React to other agents' events
   ```

3. **State management** (orchestrator.py)
   ```python
   class OrchestratorState:
       messages: List[Message]  # All agent communications
       briefing_context: Dict   # Shared context for current workflow
       conflicts: List[Conflict]  # Unresolved conflicts
       last_memory_write: datetime
   ```

### Frontend

Doesn't need to know about agent communication. Just:
- Calls `/agents/critic/evaluate` for critique
- Calls `/briefing/today` for daily briefing
- Subscribes to WebSocket for live updates
- Displays what backend sends

---

## Example: Full Workflow (Daily Briefing)

```
6:45 AM — Cron triggers: generate_daily_briefing(user_id)
│
├─ Orchestrator.command_agent(to="life_admin", "get_daily_inbox")
│  └─ LifeAdmin queries Notion + Calendar
│     └─ Returns: { top_3_tasks, critical_alerts }
│     └─ Writes to memory: "daily_inbox_generated"
│
├─ Orchestrator.command_agent(to="wellness", "get_recovery_score")
│  └─ Wellness reads health_snapshots
│     └─ Returns: { recovery_score: 62, recommendations: [...] }
│     └─ Writes to memory: "recovery_score_computed"
│
├─ Orchestrator.command_agent(to="finance", "get_portfolio_delta")
│  └─ Finance queries Plaid / Yahoo Finance
│     └─ Returns: { delta: +₹12500, percent: +1.2% }
│     └─ Writes to memory: "portfolio_delta_computed"
│
├─ Orchestrator.command_agent(to="content", "get_top_stories")
│  └─ Content queries feed_items
│     └─ Returns: { top_3_stories, idea_cards }
│     └─ Writes to memory: "feed_curated"
│
├─ Orchestrator.command_agent(to="relationship", "get_reach_outs")
│  └─ Relationship computes urgency scores
│     └─ Returns: { reach_outs: [3 contacts] }
│     └─ Writes to memory: "reach_outs_computed"
│
└─ Orchestrator merges all → DailyBriefing object
   └─ Writes to memory: "briefing_generated"
   └─ Sends via APNs / WebSocket to frontend
   └─ Frontend fetches /briefing/today and displays
```

