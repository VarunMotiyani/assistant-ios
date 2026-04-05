# JARVIS v2 — Complete Rebuild Guide

## What Is This?

A **complete, production-ready specification** for building JARVIS — a personal multi-agent AI operating system where agents talk to each other, learn from memory, and surface insights proactively.

**Key innovation:** Agents are autonomous but coordinated. They query each other, broadcast events, and synthesize decisions together. All thinking happens in the backend. Frontend is dumb.

---

## Start Here: Document Reading Order

### 1. **[SYSTEM_DESIGN_V2.md](./docs/SYSTEM_DESIGN_V2.md)** (15 min read)
**Read first.** High-level overview of the entire system.
- How the 3 layers work (frontend, backend, data)
- What agents are and why they're useful
- Basic architecture diagram
- Core principles

**After this, you should understand:** "Agents talk to each other through an Orchestrator"

---

### 2. **[AGENT_COMMUNICATION.md](./docs/AGENT_COMMUNICATION.md)** (20 min read)
**Read after System Design.** Deep dive into how agents talk.
- 4 message types (query, event, command, proposal)
- Each agent's inputs, outputs, and responsibilities
- Full workflow examples
- Conflict resolution

**After this, you should understand:** "Critic asks Wellness for health status, then uses that in its critique"

---

### 3. **[MEMORY_SYSTEM.md](./docs/MEMORY_SYSTEM.md)** (25 min read)
**Read after Agent Communication.** How agents remember and learn.
- 4-tier memory architecture (short-term, long-term, semantic, task-specific)
- What data goes where
- Memory write triggers
- Query patterns

**After this, you should understand:** "Every interaction creates a memory record that agents can later retrieve"

---

### 4. **[BACKEND_API.md](./docs/BACKEND_API.md)** (20 min read)
**Read after Memory System.** Complete API specification.
- All REST endpoints
- Request/response formats
- Authentication
- WebSocket
- Rate limiting

**After this, you should understand:** "Frontend calls GET /briefing/today and gets all agent data"

---

### 5. **[FRONTEND_SPEC.md](./docs/FRONTEND_SPEC.md)** (15 min read)
**Read after Backend API.** What the frontend does (and doesn't do).
- Key screens
- API calls per screen
- State management
- Offline-first caching
- Tech stack options

**After this, you should understand:** "Frontend is simple. Backend is smart."

---

### 6. **[IMPLEMENTATION_GUIDE.md](./docs/IMPLEMENTATION_GUIDE.md)** (Start coding)
**Read when you're ready to build.** Step-by-step instructions.
- Phase 1: Foundation (database, auth, basic agents)
- Phase 2: Real data (connect to PostgreSQL, APIs)
- Phase 3: Inter-agent communication (agents querying each other)
- Phase 4: Daily briefing (all agents work together)
- Phase 5+: Frontend, memory, deployment

**After this, you'll have:** Working code you can test locally

---

### 7. **[QUICK_REFERENCE.md](./docs/QUICK_REFERENCE.md)** (Keep handy)
**Reference while coding.** One-page cheat sheet.
- Architecture summary
- All 6 agents at a glance
- API endpoints
- Database tables
- Common debugging tips

---

## If You Only Have 5 Minutes

Read this:

```
JARVIS = 6 autonomous agents + 1 Orchestrator + 4-tier memory system

Agents:
  - LifeAdmin: Tasks, calendars, bills
  - Wellness: Sleep, workouts, recovery, mood
  - Finance: Portfolio, cashflow, savings
  - Content: News feed, idea cards
  - Relationships: People CRM, birthdays
  - Critic: Brutally honest feedback on ideas

Communication:
  - Agents ask each other questions (query)
  - Agents broadcast events (event)
  - Orchestrator commands agents (command)

Memory:
  - Short-term (Redis): Today's state
  - Long-term (PostgreSQL): Historical events
  - Semantic (pgvector): Embeddings for learning
  - Task (PostgreSQL): Agent-specific derived state

Architecture:
  Frontend (React/SwiftUI) ← REST API ← Backend (FastAPI) ← Agents + Memory

Build timeline: 10 weeks from zero to App Store
```

---

## Absolute Essentials (TL;DR)

### What Agents Do
1. **LifeAdmin:** "You have 3 critical tasks. Bill due in 2 days."
2. **Wellness:** "Recovery score 62. Recommendation: moderate workout."
3. **Finance:** "Portfolio up 1.2%. Savings rate: 38%."
4. **Content:** "Top 3 AI news items curated for you."
5. **Relationships:** "Contact Rahul (18 days overdue). Birthday coming in 5 days."
6. **Critic:** "You should launch the course. Here's why it'll fail. Here's how to fix it."

### How They Work Together
User asks Critic: "Should I build a course?"

Critic → asks Wellness: "Is user healthy?"
Critic → asks Finance: "Can user afford it?"
Critic → asks LifeAdmin: "Is user overloaded?"
Critic → fetches past similar ideas from memory

Critic synthesizes → gives verdict

### What Frontend Does
1. Fetches `/briefing/today` → shows dashboard
2. Shows tabs for each agent
3. User clicks "Ask Critic" → sends `/agents/critic/evaluate`
4. Shows response
5. Caches data locally for offline

### What Backend Does
1. Routes agent messages through Orchestrator
2. Fetches data from integrations (Google, Notion, Plaid, etc.)
3. Calls Claude for reasoning
4. Stores everything in memory
5. Pushes notifications

---

## Technology Choices

### Backend
- **Framework:** FastAPI (Python)
- **Orchestration:** LangGraph
- **Database:** PostgreSQL 16 + pgvector
- **Cache:** Redis
- **LLM:** Anthropic Claude (opus for reasoning, haiku for speed)

### Frontend (Choose One)
- **Option A:** React (fastest, most flexible)
- **Option B:** Vue (similar to React but simpler)
- **Option C:** SwiftUI (native iOS)

### Deployment
- **Backend:** AWS ECS Fargate, Heroku, Railway, or similar
- **Frontend:** Vercel, Netlify, or AWS S3 + CloudFront
- **Database:** AWS RDS PostgreSQL or cloud Postgres (Supabase, Neon)
- **Cache:** AWS ElastiCache or Redis Cloud

---

## Quick Start (Local Development)

```bash
# 1. Clone repo (or create new structure)
mkdir jarvis && cd jarvis

# 2. Backend setup
mkdir backend
cd backend
python -m venv venv
source venv/bin/activate
pip install fastapi uvicorn asyncpg aioredis pydantic pyjwt anthropic langgraph

# Create .env
echo "DATABASE_URL=postgresql://user:pass@localhost:5432/jarvis" > .env
echo "REDIS_URL=redis://localhost:6379" >> .env
echo "ANTHROPIC_API_KEY=sk-..." >> .env

# Start PostgreSQL
docker run -d -p 5432:5432 -e POSTGRES_PASSWORD=postgres postgres:16

# Start Redis
docker run -d -p 6379:6379 redis

# Run backend
python main.py
# Visit http://localhost:8000/docs

# 3. Frontend setup (if React)
cd ../frontend
npx create-react-app .
npm install axios zustand
npm start
# Visit http://localhost:3000
```

---

## Key Concepts Explained Simply

### The Orchestrator
Think of it as a **message router for agents**. When Critic wants to know the user's recovery score, it doesn't call Wellness directly. It sends a message to the Orchestrator: "Hey, ask Wellness about recovery score." The Orchestrator delivers the message, waits for a response, and returns it to Critic.

### Agent Communication Patterns

**Pattern 1: Query**
```
Critic: "Hey Orchestrator, ask Wellness: is user healthy?"
Orchestrator: [delivers message to Wellness]
Wellness: [fetches data, returns answer]
Orchestrator: [returns answer to Critic]
Critic: [uses answer in reasoning]
```

**Pattern 2: Event**
```
Wellness: "User completed workout!"
Orchestrator: [records in memory] [notifies all agents]
LifeAdmin: [notes user is less available]
Finance: [notes user is active]
Critic: [notes for future decisions]
```

### Memory Layers

**Short-term (Redis):**
Today's briefing, current health data, real-time state. Thrown away after 24 hours.

**Long-term (PostgreSQL):**
Every event ever. Workouts, tasks completed, money earned. Forever.

**Semantic (pgvector):**
Embeddings of past decisions. "I've seen this type of idea before..." Uses AI similarity search.

**Task-specific (PostgreSQL):**
Computed facts. Workout streak = 8 days. Mood trend = improving. Emergency fund = 5.2 months.

---

## What Makes This Different

### vs. ChatGPT
- ✅ Agents are **proactive** (nudges you), not reactive (you chat)
- ✅ **Multi-agent** (6 agents debating), not single AI
- ✅ **Integrated data** (knows your health, finances, relationships), not general knowledge
- ✅ **Context-aware** (remembers everything), not stateless
- ✅ **Your data only** (self-hosted option), not sent to OpenAI

### vs. Traditional SaaS
- ✅ Agents talk to **each other**, not isolated modules
- ✅ **Learns over time** (semantic memory), not static
- ✅ **One system** (JARVIS), not 10 apps
- ✅ **Actually useful** (surfaces insights), not just displays data

---

## Realistic Scope & Timeline

### 10-Week Plan

| Week | Phase | Work | Deliverable |
|---|---|---|---|
| 1 | Foundation | Database, auth, Orchestrator skeleton | Backend receives requests |
| 2 | Real data | Connect agents to DB/APIs | Agents read real data |
| 3 | Communication | Inter-agent queries, LLM integration | Critic queries other agents |
| 4 | Orchestration | Daily briefing workflow, scheduling | All agents work together |
| 5 | Frontend | Home screen, tabs, styling | Can see data |
| 6 | Memory | Semantic search, learning | Agents improve with time |
| 7 | Polish | Error handling, logging, tests | Production-quality code |
| 8 | Deploy | Backend + frontend live | Working on internet |
| 9 | iOS | Native app (optional) | HealthKit integration, widgets |
| 10 | Launch | App Store submission | Public release |

**Solo developer:** 10 weeks is aggressive. Plan 15-20 weeks. Outsource iOS if you want.

---

## Not Included (Save for Later)

- 🤖 Multi-user support (build single-user first)
- 📱 Apple Watch complications (Phase 2+)
- 🔔 Push notifications via APNs (Phase 1.5)
- 🌐 Multi-device sync (Phase 2+)
- 💬 Chat interface (Core feature is insights, not chat)
- 🎯 Advanced prompt engineering (Start simple, iterate)

---

## Common Mistakes to Avoid

❌ **Mistake:** Building all 6 agents before testing basic communication
✅ **Do this:** Build Wellness + Critic first. Get query/response working. Then add others.

❌ **Mistake:** Storing everything in cache (Redis)
✅ **Do this:** Cache today's data only. Store everything in PostgreSQL. Replay from DB as needed.

❌ **Mistake:** Having frontend call integrations (Google, Notion, Plaid)
✅ **Do this:** Backend calls integrations. Frontend only talks to your API.

❌ **Mistake:** Hardcoding API keys in code
✅ **Do this:** Use environment variables. Use .env locally. Use secrets manager in production.

❌ **Mistake:** Building beautiful UI before agents work
✅ **Do this:** Make agents work first. UI can be ugly initially. Polish later.

---

## How to Use This Documentation

### For Building Code
1. Read IMPLEMENTATION_GUIDE (has actual code)
2. Refer to BACKEND_API for endpoint details
3. Refer to FRONTEND_SPEC for UI details
4. Keep QUICK_REFERENCE.md open while coding

### For Understanding Architecture
1. Start with SYSTEM_DESIGN_V2
2. Deep dive: AGENT_COMMUNICATION + MEMORY_SYSTEM
3. Reference: QUICK_REFERENCE

### For Debugging
1. Check logs (backend output)
2. Check database (psql)
3. Check browser network tab (frontend)
4. Refer to QUICK_REFERENCE debugging section
5. Refer to IMPLEMENTATION_GUIDE troubleshooting

---

## Next Steps

### Right Now
1. ✅ Read SYSTEM_DESIGN_V2.md
2. ✅ Read AGENT_COMMUNICATION.md
3. ✅ Skim MEMORY_SYSTEM.md and BACKEND_API.md

### Tomorrow
1. Set up local development environment (PostgreSQL, Redis, Python)
2. Follow Phase 1 of IMPLEMENTATION_GUIDE
3. Get basic backend running

### This Week
1. Implement 2-3 agents (Wellness, Finance, LifeAdmin)
2. Get Orchestrator routing messages
3. Test with `curl` commands

### This Month
1. Add real data sources (Google Calendar, Notion, Plaid)
2. Build frontend home screen
3. Deploy to cloud
4. Add more agents

---

## Getting Help

### Debugging
- 📖 Check QUICK_REFERENCE.md "Debugging Tips" section
- 📖 Check IMPLEMENTATION_GUIDE.md "Troubleshooting Checklist"
- 🔍 Check logs: `docker logs <container_id>` (backend), DevTools Network (frontend)
- 🗄️ Check database: `psql -d jarvis` then `SELECT ...`

### Architecture Questions
- 📖 SYSTEM_DESIGN_V2.md → high-level overview
- 📖 AGENT_COMMUNICATION.md → how agents interact
- 📖 MEMORY_SYSTEM.md → how memory works

### Code Questions
- 📖 BACKEND_API.md → endpoint specifications
- 📖 IMPLEMENTATION_GUIDE.md → step-by-step code
- 📖 QUICK_REFERENCE.md → snippets and patterns

---

## Success Metrics

### Week 1
- [ ] Backend starts without errors
- [ ] Database connected
- [ ] Auth middleware works
- [ ] Orchestrator routes messages

### Week 4
- [ ] Daily briefing workflow runs
- [ ] All 6 agents respond to queries
- [ ] Memory stores events
- [ ] Can run `curl /briefing/today` and get complete response

### Week 8
- [ ] Frontend deployed
- [ ] Can see home screen
- [ ] All tabs functional
- [ ] Offline caching works
- [ ] Notifications fire

### Week 10
- [ ] App Store submission
- [ ] TestFlight beta working
- [ ] Zero critical bugs
- [ ] Demo video ready

---

## Files in This Documentation

```
docs/
├── SYSTEM_DESIGN_V2.md         ← START HERE (15 min)
├── AGENT_COMMUNICATION.md      ← Then read this (20 min)
├── MEMORY_SYSTEM.md            ← Then read this (25 min)
├── BACKEND_API.md              ← Complete API reference (20 min)
├── FRONTEND_SPEC.md            ← Frontend guide (15 min)
├── IMPLEMENTATION_GUIDE.md     ← Start coding here
└── QUICK_REFERENCE.md          ← Keep this handy while coding

README_V2.md (this file)         ← You are here
```

---

## License & Attribution

This is **your project**. Build on top of this. Modify, extend, remove. This documentation is a starting point, not a constraint.

The core insight: **Agents as autonomous but coordinated nodes, communicating through an orchestrator with rich persistent memory.** This pattern scales and is production-ready.

---

## Let's Build

You have:
- ✅ Clear architecture
- ✅ Detailed specifications
- ✅ Implementation guide
- ✅ API reference
- ✅ Step-by-step instructions

You're missing only code in your editor and time. Start with SYSTEM_DESIGN_V2.md. You've got this.

**Questions or stuck?** Refer to the appropriate doc. If still stuck, simplify and build the smallest possible version first.

**Good luck! 🚀**

