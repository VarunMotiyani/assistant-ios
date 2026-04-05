# JARVIS v2 — Complete System Design

## Vision

JARVIS is a **personal multi-agent AI OS** that:
- Runs 6 specialized agents that think independently
- Agents **communicate with each other** to make better decisions
- A central **Orchestrator** coordinates execution and memory
- **Rich memory system** lets agents learn from every interaction
- Surfaces insights via **notifications, API, and simple web/mobile UI**

This is **not a chatbot**. It's an autonomous system that:
- Monitors your calendar, health, finances, relationships, content, and decisions
- Agents debate internally and reach consensus
- Proactively nudges you with context-aware insights
- Remembers everything and improves over time

---

## System Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    FRONTEND LAYER (Browser + iOS)                │
│                                                                   │
│  ┌─────────────────┐  ┌──────────────────┐  ┌──────────────────┐
│  │   Home Screen   │  │   Agent Panels   │  │   Memory View     │
│  │   (Dashboard)   │  │   (Insights)     │  │   (History)       │
│  └─────────────────┘  └──────────────────┘  └──────────────────┘
│                                                                   │
└──────────────────────────────┬──────────────────────────────────┘
                               │
                       REST API / WebSocket
                               │
┌──────────────────────────────▼──────────────────────────────────┐
│                   BACKEND API LAYER (FastAPI)                    │
│                                                                   │
│  ┌─────────────┐  ┌──────────────┐  ┌──────────────────────┐   │
│  │   Routes    │  │ Middleware   │  │  WebSocket Manager   │   │
│  │   /agents   │  │  (Auth, Rate │  │  (Real-time updates) │   │
│  │   /memory   │  │   limit)     │  │                      │   │
│  │   /events   │  └──────────────┘  └──────────────────────┘   │
│  └─────────────┘                                                 │
│        │                                                          │
└────────┼──────────────────────────────────────────────────────────┘
         │
┌────────▼──────────────────────────────────────────────────────────┐
│              ORCHESTRATOR LAYER (LangGraph + State)               │
│                                                                   │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  StateGraph (Message Bus + Execution Engine)             │   │
│  │                                                           │   │
│  │  - Routes messages between agents                        │   │
│  │  - Manages shared conversation state                     │   │
│  │  - Triggers memory writes                                │   │
│  │  - Coordinates multi-agent workflows                     │   │
│  │  - Handles conflicts and consensus                       │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                   │
└────────┬──────────────────────────────────────────────────────────┘
         │
    ┌────┴────────────────────────────────────────┬─────────────┐
    │                                              │             │
┌───▼──────────────────┐  ┌──────────────────┐  ┌─▼──────────┐ │
│  AGENT LAYER         │  │  MEMORY LAYER    │  │ INTEGRATIO │ │
│                      │  │                  │  │ NS LAYER   │ │
│ ┌──────────────────┐ │  │ ┌─────────────┐ │  │            │ │
│ │ LifeAdmin Agent  │ │  │ │ Short-term  │ │  │ ┌────────┐ │ │
│ │ Wellness Agent   │ │  │ │ (Redis)     │ │  │ │Google  │ │ │
│ │ Finance Agent    │ │  │ │             │ │  │ │Calendar│ │ │
│ │ Content Agent    │ │  │ │ ┌─────────┐ │ │  │ └────────┘ │ │
│ │ Relationship Agn │ │  │ │ │Long-term│ │ │  │ ┌────────┐ │ │
│ │ Critic Agent     │ │  │ │ │(Postgres)│ │ │  │ │Notion  │ │ │
│ └──────────────────┘ │  │ │ └─────────┘ │ │  │ └────────┘ │ │
│                      │  │ │             │ │  │ ┌────────┐ │ │
│ Each agent:          │  │ │ ┌─────────┐ │ │  │ │Plaid / │ │ │
│ - Has tools          │  │ │ │Semantic │ │ │  │ │Finance │ │ │
│ - Maintains state    │  │ │ │(pgvector)│ │ │  │ └────────┘ │ │
│ - Can message others │  │ │ └─────────┘ │ │  │            │ │
│ - Reads from memory  │  │ │             │ │  │ ┌────────┐ │ │
│ - Writes to memory   │  │ │ ┌─────────┐ │ │  │ │HealthKit   │ │
│                      │  │ │ │Task     │ │ │  │ │       │ │ │
│                      │  │ │ │Memory   │ │ │  │ └────────┘ │ │
│                      │  │ │ └─────────┘ │ │  │            │ │
│                      │  │ │             │ │  │ + News APIs│ │
│                      │  │ │ ┌─────────┐ │ │  │ + Smallcase│ │
│                      │  │ │ │Contact  │ │ │  │ + Kite API │ │
│                      │  │ │ │Network  │ │ │  │            │ │
│                      │  │ │ └─────────┘ │ │  │            │ │
│                      │  │               │ │  │            │ │
│                      │  └───────────────┘ │  └────────────┘ │
│                      │                     │                 │
└──────────────────────┘                     └─────────────────┘
         │                                              │
         └──────────────────────┬───────────────────────┘
                                │
                   ┌────────────▼──────────────┐
                   │   DATA LAYER             │
                   │                          │
                   │  PostgreSQL + pgvector   │
                   │  Redis (ephemeral state) │
                   │                          │
                   └──────────────────────────┘
```

---

## Core Concepts

### 1. **Agents Are Autonomous Nodes**

Each agent:
- Has a **distinct expertise** (life admin, wellness, etc.)
- Maintains its own **local state** (what it cares about)
- Can **ask other agents** for information
- Can **propose actions** to the orchestrator
- Can **read and write to shared memory**

Example flow:
```
User: "Should I take a startup idea to market?"
├─ Critic Agent analyzes idea
├─ Asks Wellness: "Is user recovered enough to handle startup stress?"
├─ Asks Finance: "Can user sustain 3 months unpaid work?"
├─ Asks LifeAdmin: "What's user's calendar load?"
├─ Agents report back
└─ Critic synthesizes → verdict + reasoning
```

### 2. **Orchestrator as Message Bus**

The Orchestrator (LangGraph StateGraph):
- Receives messages from agents
- Routes messages to other agents
- Maintains shared conversation state
- Triggers memory updates
- Coordinates workflows (e.g., "daily briefing" workflow calls all agents)
- Handles inter-agent conflicts (two agents propose conflicting actions)

### 3. **Memory System (Multi-Layer)**

**Short-term (Redis, 24h TTL)**
- Current conversation context
- Today's briefing data
- Real-time state (what user just did)

**Long-term (PostgreSQL)**
- All historical events (tasks, workouts, trades, contacts)
- User preferences
- One entry per fact (normalized)

**Semantic (pgvector)**
- Embeddings of past critic sessions, insights, decisions
- Used for similarity search ("similar ideas I've evaluated before")
- Supports semantic memory retrieval

**Task Memory (PostgreSQL)**
- Agent-specific state (e.g., Wellness agent's "workout streak")
- Used to spot patterns and trends

**Contact Network (Graph-like in PostgreSQL)**
- People, relationships, interaction history
- Used by Relationship agent for reach-out decisions

---

## Communication Patterns

### Pattern 1: Agent Queries Orchestrator

Agent asks: "What's the user's recovery score?"

```python
# Inside an agent
health_data = await orchestrator.query("wellness", "get_recovery_score")
# Orchestrator finds latest health_snapshot in memory, returns it
```

### Pattern 2: Agent Messages Another Agent

Agent asks: "Can user handle startup workload?"

```python
# Inside Critic Agent
response = await orchestrator.send_to_agent(
    to="wellness",
    message="Is user recovered enough for high-stress work?",
    context={"startup_idea": "..."}
)
# Wellness Agent processes, returns verdict
```

### Pattern 3: Agent Broadcasts to All

Agent publishes: "User completed workout"

```python
await orchestrator.broadcast("event", {
    "type": "workout_completed",
    "workout_type": "strength",
    "duration": 60,
    "calories": 400
})
# All agents can react to this event
# Memory system auto-records it
```

### Pattern 4: Workflow Coordination

Orchestrator triggers: "Generate daily briefing"

```python
# Orchestrator calls workflow
briefing = await orchestrator.run_workflow("daily_briefing", {
    "user_id": "...",
    "date": "2026-04-05"
})
# Workflow:
# 1. LifeAdmin Agent → top 3 tasks
# 2. Wellness Agent → recovery score + recommendation
# 3. Finance Agent → portfolio delta
# 4. Content Agent → top 3 news items
# 5. Relationship Agent → who to contact today
# 6. Orchestrator merges results → briefing object
```

---

## Data Flow Examples

### Example 1: Morning Routine

```
6:45 AM — Cron job triggers "generate_daily_briefing" workflow
│
├─ Step 1: Orchestrator tells LifeAdmin
│  └─ LifeAdmin fetches Notion tasks + Google Calendar
│     └─ Returns: { top_3_tasks, critical_alerts, prep_notes }
│
├─ Step 2: Orchestrator tells Wellness
│  └─ Wellness reads health_snapshots from memory
│     └─ Returns: { recovery_score, sleep_quality, workout_rec }
│
├─ Step 3: Orchestrator tells Finance
│  └─ Finance reads holdings from memory
│     └─ Returns: { portfolio_delta, cash_position, trend }
│
├─ Step 4: Orchestrator tells Content
│  └─ Content fetches latest feed items
│     └─ Returns: { top_3_stories, idea_cards }
│
├─ Step 5: Orchestrator tells Relationship
│  └─ Relationship reads contacts from memory
│     └─ Returns: { reach_outs_today, upcoming_moments }
│
└─ Step 6: Orchestrator synthesizes all → DailyBriefing object
   └─ Writes to memory
   └─ Sends push notification to frontend
   └─ Frontend fetches via /briefing/today
```

### Example 2: User Asks Critic

```
User (via mobile): "Critique my idea to build an AI course"
│
├─ Request: POST /agents/critic/evaluate
│  { prompt: "Should I launch an AI course?" }
│
├─ Frontend sends to Backend API
│  └─ Backend routes to Orchestrator
│
├─ Orchestrator executes Critic workflow:
│  │
│  ├─ Critic reads user prompt
│  │
│  ├─ Critic asks Wellness: "User's recovery score?"
│  │  └─ Wellness responds: 62/100 (moderate)
│  │
│  ├─ Critic asks Finance: "Can user sustain 3 months?"
│  │  └─ Finance responds: { runway: 8_months, savings_rate: 35% }
│  │
│  ├─ Critic asks LifeAdmin: "Calendar load?"
│  │  └─ LifeAdmin responds: { meetings_per_week: 12, overdue_tasks: 3 }
│  │
│  ├─ Critic queries memory: "Similar ideas I've evaluated?"
│  │  └─ Memory returns: [{ idea: "course_idea_6mo_ago", verdict: "red" }]
│  │
│  ├─ Critic uses all context to form critique
│  │
│  └─ Orchestrator writes session to memory (with embedding)
│
└─ Frontend receives:
   { 
     verdict: "yellow",
     core_critique: [...],
     assumptions: [...],
     missing_info: [...],
     if_you_must: [...]
   }
```

---

## Frontend vs Backend Responsibilities

### Backend Owns:
- ✅ All LLM calls
- ✅ Agent orchestration
- ✅ Memory system
- ✅ Integration with external APIs (Notion, Google Calendar, Plaid, HealthKit sync)
- ✅ All business logic
- ✅ Cron jobs and background tasks
- ✅ Data persistence

### Frontend Owns:
- ✅ User interface (rendering)
- ✅ User input (forms, voice, swipe gestures)
- ✅ Local app state (UI state, animations)
- ✅ Offline-first caching (SwiftData on iOS, localStorage on web)
- ✅ Requesting data from backend
- ✅ Displaying backend responses
- ✅ HealthKit permission requests (iOS only) + syncing to backend

### Clear Rule:
**Frontend is dumb. Backend is smart.**

---

## What This Enables

1. **Agents talk to each other** → Better decisions
2. **Rich memory** → Context-aware insights
3. **Easy to extend** → Add new agents without touching frontend
4. **Scalable** → Backend can run on servers, frontend is thin
5. **Offline-ready** → Frontend caches last-known data
6. **Real-time** → WebSocket for live updates
7. **Debuggable** → All agent reasoning is logged in memory

---

## Non-Negotiables

✅ **All AI reasoning on backend**
✅ **Agents are autonomous but coordinated**
✅ **Memory is the source of truth**
✅ **Frontend is simple and stateless**
✅ **Every interaction creates a memory record**
✅ **Security: secrets in env vars, no hardcoding**
✅ **Production-grade: proper error handling, logging, retry logic**

