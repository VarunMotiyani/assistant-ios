# JARVIS Implementation Guide

## How to Build This from Scratch

This guide tells you **exactly what to build, in what order, and why**.

---

## Phase 1: Foundation (Week 1)

### Goal
Get the basic infrastructure working so agents can exist and talk.

### What to Build

#### 1.1 Project Setup
```bash
# Backend
mkdir jarvis-backend && cd jarvis-backend
python -m venv venv
source venv/bin/activate
pip install fastapi uvicorn psycopg2-binary asyncpg aioredis python-dotenv pydantic pyjwt anthropic langgraph langchain

# Create .env file
cat > .env << EOF
DATABASE_URL=postgresql://user:pass@localhost:5432/jarvis
REDIS_URL=redis://localhost:6379
ANTHROPIC_API_KEY=sk-...
JWT_SECRET_KEY=your-secret-key-here
EOF

# Frontend (if going web)
npm create react-app jarvis-frontend
cd jarvis-frontend
npm install axios zustand
```

#### 1.2 Database Setup
```bash
# Install PostgreSQL locally or use cloud (e.g., Supabase)
# Install pgvector extension

psql -U postgres -d jarvis << EOF
CREATE EXTENSION vector;

-- Users table
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email TEXT UNIQUE NOT NULL,
  name TEXT,
  created_at TIMESTAMPTZ DEFAULT now()
);

-- User events (master log)
CREATE TABLE user_events (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id),
  event_type TEXT NOT NULL,
  data JSONB,
  agent_source TEXT,
  created_at TIMESTAMPTZ DEFAULT now()
);
CREATE INDEX idx_user_events ON user_events(user_id, created_at DESC);

-- Agent communications (audit trail)
CREATE TABLE agent_communications (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id),
  from_agent TEXT,
  to_agent TEXT,
  message_type TEXT, -- query, event, command, proposal
  payload JSONB,
  created_at TIMESTAMPTZ DEFAULT now()
);

-- Simple preferences
CREATE TABLE user_preferences (
  user_id UUID REFERENCES users(id),
  key TEXT,
  value JSONB,
  PRIMARY KEY (user_id, key)
);
EOF
```

#### 1.3 Backend Main Structure
```python
# backend/main.py
from fastapi import FastAPI, Depends, HTTPException
from fastapi.middleware.cors import CORSMiddleware
import os
from dotenv import load_dotenv

load_dotenv()

app = FastAPI(title="JARVIS", version="0.1.0")

# CORS
app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000", "https://yourfrontend.com"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Health check
@app.get("/health")
async def health():
    return {"status": "ok"}

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

```bash
python main.py
# Test: curl http://localhost:8000/health
```

#### 1.4 Auth Middleware
```python
# backend/auth.py
from fastapi import Depends, HTTPException, status
from fastapi.security import HTTPBearer, HTTPAuthCredentials
import jwt
from datetime import datetime, timedelta

security = HTTPBearer()

def create_token(user_id: str) -> str:
    payload = {
        "sub": user_id,
        "iat": datetime.utcnow(),
        "exp": datetime.utcnow() + timedelta(days=1)
    }
    return jwt.encode(payload, os.getenv("JWT_SECRET_KEY"), algorithm="HS256")

async def verify_token(credentials: HTTPAuthCredentials = Depends(security)):
    try:
        payload = jwt.decode(
            credentials.credentials,
            os.getenv("JWT_SECRET_KEY"),
            algorithms=["HS256"]
        )
        user_id = payload.get("sub")
        if not user_id:
            raise HTTPException(status_code=401, detail="Invalid token")
        return user_id
    except jwt.ExpiredSignatureError:
        raise HTTPException(status_code=401, detail="Token expired")
    except jwt.InvalidTokenError:
        raise HTTPException(status_code=401, detail="Invalid token")
```

#### 1.5 Orchestrator Skeleton
```python
# backend/orchestrator/orchestrator.py
from typing import Dict, Any, List
import asyncio
from datetime import datetime

class Message:
    def __init__(self, sender: str, recipient: str, msg_type: str, content: Dict):
        self.sender = sender
        self.recipient = recipient
        self.msg_type = msg_type  # query, event, command, proposal
        self.content = content
        self.timestamp = datetime.utcnow()

class Orchestrator:
    def __init__(self, memory_manager, db, redis):
        self.memory = memory_manager
        self.db = db
        self.redis = redis
        self.agents = {}
        self.messages: List[Message] = []
    
    def register_agent(self, name: str, agent):
        """Register an agent with the orchestrator"""
        self.agents[name] = agent
    
    async def query(self, from_agent: str, to_agent: str, query_type: str, context: Dict = None):
        """Agent asks another agent a question"""
        msg = Message(from_agent, to_agent, "query", {
            "query_type": query_type,
            "context": context
        })
        self.messages.append(msg)
        
        # Log to database
        await self.memory.log_communication(msg)
        
        # Call the agent
        if to_agent not in self.agents:
            raise ValueError(f"Agent {to_agent} not found")
        
        agent = self.agents[to_agent]
        response = await agent.handle_query(query_type, context)
        return response
    
    async def broadcast_event(self, from_agent: str, event_type: str, payload: Dict):
        """Agent broadcasts an event to all agents + memory"""
        msg = Message(from_agent, "all", "event", {
            "event_type": event_type,
            "payload": payload
        })
        self.messages.append(msg)
        
        # Write to memory
        await self.memory.write_event(from_agent, event_type, payload)
        
        # Notify all subscribed agents (optional)
        for agent_name, agent in self.agents.items():
            if hasattr(agent, 'handle_event'):
                try:
                    await agent.handle_event(event_type, payload)
                except Exception as e:
                    print(f"Agent {agent_name} error handling event: {e}")
    
    async def command_agent(self, to_agent: str, command: str, params: Dict = None):
        """Orchestrator tells an agent to do something"""
        agent = self.agents[to_agent]
        response = await agent.handle_command(command, params)
        return response
```

#### 1.6 Memory Manager Skeleton
```python
# backend/memory/manager.py
from datetime import datetime
import json
import asyncpg

class MemoryManager:
    def __init__(self, db, redis):
        self.db = db  # asyncpg connection pool
        self.redis = redis  # aioredis client
    
    async def write_event(self, agent_source: str, event_type: str, data: Dict):
        """Write an event to memory"""
        # Write to PostgreSQL
        await self.db.execute(
            """
            INSERT INTO user_events (user_id, event_type, data, agent_source)
            VALUES ($1, $2, $3, $4)
            """,
            user_id, event_type, json.dumps(data), agent_source
        )
        
        # Also cache in Redis if relevant
        if event_type in ['recovery_score_computed', 'portfolio_updated']:
            await self.redis.setex(
                f"{event_type}:{user_id}",
                3600,
                json.dumps(data)
            )
    
    async def log_communication(self, message: Message):
        """Log agent-to-agent communication"""
        await self.db.execute(
            """
            INSERT INTO agent_communications (user_id, from_agent, to_agent, message_type, payload)
            VALUES ($1, $2, $3, $4, $5)
            """,
            user_id, message.sender, message.recipient, message.msg_type, 
            json.dumps(message.content)
        )
    
    async def query_facts(self, user_id: str, fact_type: str):
        """Query a fact from memory"""
        # First check Redis cache
        cached = await self.redis.get(f"fact:{user_id}:{fact_type}")
        if cached:
            return json.loads(cached)
        
        # Query database
        # (specific to fact_type)
        return None
```

#### 1.7 Base Agent Class
```python
# backend/agents/base_agent.py
from abc import ABC, abstractmethod

class BaseAgent(ABC):
    def __init__(self, name: str, orchestrator, memory):
        self.name = name
        self.orchestrator = orchestrator
        self.memory = memory
    
    @abstractmethod
    async def handle_query(self, query_type: str, context: Dict = None) -> Dict:
        """Handle a query from another agent"""
        pass
    
    @abstractmethod
    async def handle_event(self, event_type: str, payload: Dict):
        """React to an event broadcast by another agent"""
        pass
    
    async def ask_agent(self, other_agent: str, query_type: str, context: Dict = None):
        """Ask another agent a question"""
        return await self.orchestrator.query(self.name, other_agent, query_type, context)
    
    async def broadcast(self, event_type: str, payload: Dict):
        """Broadcast an event to all agents"""
        await self.orchestrator.broadcast_event(self.name, event_type, payload)
```

#### 1.8 Simple Agent Implementation
```python
# backend/agents/wellness.py
from base_agent import BaseAgent

class WellnessAgent(BaseAgent):
    def __init__(self, orchestrator, memory):
        super().__init__("wellness", orchestrator, memory)
    
    async def handle_query(self, query_type: str, context: Dict = None) -> Dict:
        if query_type == "recovery_score_today":
            # Fetch from memory
            recovery_score = 62  # Hardcode for now
            return {
                "recovery_score": recovery_score,
                "sleep": 7.2,
                "hrv": 45,
                "reasoning": "Good recovery"
            }
        return {}
    
    async def handle_event(self, event_type: str, payload: Dict):
        # React to events
        if event_type == "workout_completed":
            print(f"Wellness agent saw workout: {payload}")
    
    async def handle_command(self, command: str, params: Dict = None) -> Dict:
        if command == "get_recovery_score":
            return await self.handle_query("recovery_score_today")
        return {}
```

#### 1.9 Router Setup
```python
# backend/routers/briefing.py
from fastapi import APIRouter, Depends
from auth import verify_token

router = APIRouter(prefix="/briefing", tags=["briefing"])

@router.get("/today")
async def get_briefing(user_id: str = Depends(verify_token)):
    # For now, return dummy data
    return {
        "date": "2026-04-05",
        "life_admin": {},
        "wellness": {
            "recovery_score": 62,
            "sleep": 7.2,
            "coaching_note": "Good day, push hard"
        },
        "finance": {},
        "content": {},
        "relationships": {}
    }

# Add to main.py:
from routers import briefing
app.include_router(briefing.router)
```

### Why This Phase?
- ✅ Backend can receive requests
- ✅ Orchestrator can route messages
- ✅ Agents can be created
- ✅ Memory system skeleton exists
- ✅ Auth works

### Test It
```bash
# Start backend
python backend/main.py

# Test health
curl http://localhost:8000/health

# Test briefing (no auth for now)
curl http://localhost:8000/v1/briefing/today
```

---

## Phase 2: Connect to Real Data (Week 2)

### Goal
Agents can fetch real data from external APIs and memory.

### What to Build

#### 2.1 Connect to PostgreSQL
```python
# backend/database.py
import asyncpg
import os

class Database:
    def __init__(self):
        self.pool = None
    
    async def connect(self):
        self.pool = await asyncpg.create_pool(
            dsn=os.getenv("DATABASE_URL"),
            min_size=10,
            max_size=20
        )
    
    async def disconnect(self):
        await self.pool.close()
    
    async def execute(self, query, *args):
        async with self.pool.acquire() as conn:
            return await conn.execute(query, *args)
    
    async def fetch(self, query, *args):
        async with self.pool.acquire() as conn:
            return await conn.fetch(query, *args)
    
    async def fetchval(self, query, *args):
        async with self.pool.acquire() as conn:
            return await conn.fetchval(query, *args)

# In main.py:
db = Database()

@app.on_event("startup")
async def startup():
    await db.connect()

@app.on_event("shutdown")
async def shutdown():
    await db.disconnect()
```

#### 2.2 Implement Wellness Agent with Real Data
```python
# backend/agents/wellness.py
class WellnessAgent(BaseAgent):
    async def handle_query(self, query_type: str, context: Dict = None) -> Dict:
        if query_type == "recovery_score_today":
            # Fetch from memory
            snapshot = await self.memory.db.fetchrow(
                """
                SELECT recovery_score, sleep_duration_minutes, hrv_ms, resting_hr_bpm, mood
                FROM wellness_snapshots
                WHERE user_id = $1 AND date = CURRENT_DATE
                """,
                context.get("user_id")
            )
            
            if snapshot:
                return {
                    "recovery_score": snapshot['recovery_score'],
                    "sleep": snapshot['sleep_duration_minutes'] / 60,
                    "hrv": snapshot['hrv_ms'],
                    "resting_hr": snapshot['resting_hr_bpm'],
                    "mood": snapshot['mood'],
                    "reasoning": self._compute_reasoning(snapshot)
                }
            return {"recovery_score": 0, "reasoning": "No data yet"}
    
    def _compute_reasoning(self, snapshot):
        if snapshot['recovery_score'] >= 70:
            return "Great recovery, you can push hard today"
        elif snapshot['recovery_score'] >= 50:
            return "Moderate recovery, pace yourself"
        else:
            return "Low recovery, consider rest or light activity"
```

#### 2.3 Implement Finance Agent
```python
# backend/agents/finance.py
class FinanceAgent(BaseAgent):
    async def handle_query(self, query_type: str, context: Dict = None) -> Dict:
        user_id = context.get("user_id")
        
        if query_type == "portfolio_value":
            holdings = await self.memory.db.fetch(
                """
                SELECT symbol, current_value FROM holdings WHERE user_id = $1
                """,
                user_id
            )
            total = sum(h['current_value'] for h in holdings)
            return {
                "portfolio_value": total,
                "currency": "INR",
                "holdings_count": len(holdings)
            }
        
        return {}
```

#### 2.4 HealthKit Sync (iOS only)
Create a simple endpoint that iOS app calls to sync HealthKit data:

```python
# backend/routers/health.py
from fastapi import APIRouter, Depends, Body
from auth import verify_token

router = APIRouter(prefix="/health", tags=["health"])

@router.post("/sync")
async def sync_health_data(
    user_id: str = Depends(verify_token),
    data: Dict = Body(...)
):
    """iOS app sends HealthKit data here"""
    # data contains: sleep_data, workout_data, steps, etc.
    
    # Save to database
    await db.execute(
        """
        INSERT INTO wellness_snapshots 
        (user_id, date, sleep_duration_minutes, sleep_efficiency_pct, hrv_ms, resting_hr_bpm, steps)
        VALUES ($1, CURRENT_DATE, $2, $3, $4, $5, $6)
        ON CONFLICT (user_id, date) DO UPDATE SET
        sleep_duration_minutes = EXCLUDED.sleep_duration_minutes,
        ...
        """,
        user_id, data['sleep_minutes'], data['sleep_efficiency'], ...
    )
    
    # Broadcast event so other agents react
    await orchestrator.broadcast_event("health_sync", "health_synced", {
        "user_id": user_id,
        "sleep_minutes": data['sleep_minutes'],
        "timestamp": datetime.utcnow().isoformat()
    })
    
    return {"ok": True}
```

#### 2.5 Frontend: Fetch and Display
```jsx
// frontend/src/pages/Home.jsx
import { useEffect, useState } from 'react';
import apiClient from '../api/client';

export default function Home() {
  const [briefing, setBriefing] = useState(null);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    fetchBriefing();
  }, []);
  
  async function fetchBriefing() {
    try {
      const data = await apiClient.get('/briefing/today');
      setBriefing(data);
    } catch (error) {
      console.error(error);
    } finally {
      setLoading(false);
    }
  }
  
  if (loading) return <div>Loading...</div>;
  
  return (
    <div>
      <h1>Today's Briefing</h1>
      <div>Recovery Score: {briefing.wellness.recovery_score}/100</div>
      <div>Portfolio: ₹{briefing.finance.portfolio_value}</div>
    </div>
  );
}
```

### Test It
```bash
# Seed some test data
psql -U postgres -d jarvis << EOF
INSERT INTO users (id, email) VALUES ('123e4567-e89b-12d3-a456-426614174000', 'test@example.com');
INSERT INTO wellness_snapshots (user_id, date, sleep_duration_minutes, recovery_score)
VALUES ('123e4567-e89b-12d3-a456-426614174000', CURRENT_DATE, 432, 62);
EOF

# Call API with test user ID in header
curl -H "Authorization: Bearer <test_token>" http://localhost:8000/v1/briefing/today
```

---

## Phase 3: Agents Querying Each Other (Week 3)

### Goal
Critic Agent can ask other agents questions and synthesize responses.

### What to Build

#### 3.1 Implement Critic Agent
```python
# backend/agents/critic.py
from base_agent import BaseAgent
from anthropic import Anthropic

class CriticAgent(BaseAgent):
    def __init__(self, orchestrator, memory, anthropic_client):
        super().__init__("critic", orchestrator, memory)
        self.client = anthropic_client
    
    async def handle_command(self, command: str, params: Dict) -> Dict:
        if command == "evaluate":
            return await self.evaluate_idea(params)
        return {}
    
    async def evaluate_idea(self, params: Dict) -> Dict:
        user_id = params.get("user_id")
        prompt = params.get("prompt")
        inject_context = params.get("inject_context", True)
        
        # Gather context from other agents
        context = {}
        if inject_context:
            # Ask Wellness
            wellness = await self.ask_agent("wellness", "recovery_score_today", {"user_id": user_id})
            context["wellness"] = wellness
            
            # Ask Finance
            finance = await self.ask_agent("finance", "portfolio_value", {"user_id": user_id})
            context["finance"] = finance
            
            # Ask LifeAdmin
            admin = await self.ask_agent("life_admin", "daily_inbox", {"user_id": user_id})
            context["life_admin"] = admin
        
        # Call Claude with context
        system_prompt = """You are a brutally honest critic. Your job is to identify every flaw, risk, and weak assumption. 
        Then tell the user what they need to do to make it work.
        
        Output a JSON object with:
        - verdict: "red" (don't do), "yellow" (risky), "green" (do it)
        - core_critique: list of 3-5 specific problems
        - assumptions: what's risky
        - missing_info: what to research
        - if_you_must_proceed: 3 changes to improve
        - personal_context: how this fits their life right now
        """
        
        user_message = f"""Critique this idea:
        {prompt}
        
        User context:
        {json.dumps(context, indent=2)}
        """
        
        response = self.client.messages.create(
            model="claude-opus-4",
            max_tokens=2000,
            system=system_prompt,
            messages=[{"role": "user", "content": user_message}]
        )
        
        critique_text = response.content[0].text
        critique_obj = json.loads(critique_text)
        
        # Save to memory
        await self.memory.write_event(
            "critic", "critique_session",
            {
                "prompt": prompt,
                "verdict": critique_obj["verdict"],
                "critique": critique_obj
            }
        )
        
        return critique_obj
```

#### 3.2 API Endpoint
```python
# backend/routers/agents.py
from fastapi import APIRouter, Depends
from auth import verify_token
from orchestrator import orchestrator

router = APIRouter(prefix="/agents", tags=["agents"])

@router.post("/critic/evaluate")
async def evaluate_idea(
    user_id: str = Depends(verify_token),
    prompt: str = None,
    inject_context: bool = True
):
    critique = await orchestrator.command_agent(
        "critic",
        "evaluate",
        {
            "user_id": user_id,
            "prompt": prompt,
            "inject_context": inject_context
        }
    )
    return critique
```

### Why This Phase?
- ✅ Agents can ask other agents for context
- ✅ Critic can synthesize multi-agent input
- ✅ LLM integration works

---

## Phase 4: Daily Briefing Workflow (Week 4)

### Goal
All agents work together to generate a comprehensive daily briefing.

### What to Build

#### 4.1 Briefing Workflow
```python
# backend/orchestrator/workflows.py
async def generate_daily_briefing(orchestrator, user_id: str) -> Dict:
    """Orchestrate all agents to generate daily briefing"""
    
    # Call each agent in parallel
    life_admin = await orchestrator.command_agent("life_admin", "get_daily_inbox", {"user_id": user_id})
    wellness = await orchestrator.command_agent("wellness", "get_recovery_score", {"user_id": user_id})
    finance = await orchestrator.command_agent("finance", "get_portfolio_delta", {"user_id": user_id})
    content = await orchestrator.command_agent("content", "get_top_stories", {"user_id": user_id})
    relationships = await orchestrator.command_agent("relationships", "get_reach_outs", {"user_id": user_id})
    
    # Merge into briefing
    briefing = {
        "date": str(date.today()),
        "generated_at": datetime.utcnow().isoformat(),
        "life_admin": life_admin,
        "wellness": wellness,
        "finance": finance,
        "content": content,
        "relationships": relationships
    }
    
    # Save to memory
    await orchestrator.memory.write_event("orchestrator", "briefing_generated", briefing)
    
    return briefing
```

#### 4.2 Cron Job
```python
# backend/tasks.py
from apscheduler.schedulers.asyncio import AsyncIOScheduler
from datetime import time

scheduler = AsyncIOScheduler()

async def daily_briefing_job(user_id: str):
    from orchestrator.workflows import generate_daily_briefing
    briefing = await generate_daily_briefing(orchestrator, user_id)
    # Push to frontend via WebSocket or APNs
    await push_to_user(user_id, briefing)

# Schedule for 6:45 AM
scheduler.add_job(
    daily_briefing_job,
    trigger="cron",
    hour=6,
    minute=45,
    args=["user_123"]
)

scheduler.start()
```

---

## Phase 5: Frontend Dashboard (Week 5)

### Goal
Beautiful frontend that displays all agent data.

### What to Build

#### 5.1 Home Screen
```jsx
// frontend/src/pages/Home.jsx
export default function Home() {
  const [briefing, setBriefing] = useState(null);
  
  useEffect(() => {
    fetchBriefing();
  }, []);
  
  async function fetchBriefing() {
    const data = await apiClient.get('/briefing/today');
    setBriefing(data);
  }
  
  return (
    <div className="home">
      <h1>Today's Briefing</h1>
      
      <Card title="Recovery Score">
        <RecoveryScoreRing score={briefing.wellness.recovery_score} />
        <p>{briefing.wellness.coaching_note}</p>
      </Card>
      
      <Card title="Top Tasks">
        {briefing.life_admin.top_3_tasks.map(task => (
          <TaskItem key={task.id} task={task} />
        ))}
      </Card>
      
      <Card title="Portfolio">
        <p>₹{briefing.finance.portfolio_value}</p>
        <p className="delta">{briefing.finance.today_delta}</p>
      </Card>
      
      <Card title="Today's News">
        {briefing.content.top_3_stories.map(story => (
          <StoryItem key={story.id} story={story} />
        ))}
      </Card>
      
      <Card title="Reach Out Today">
        {briefing.relationships.reach_outs_today.map(contact => (
          <ContactItem key={contact.id} contact={contact} />
        ))}
      </Card>
    </div>
  );
}
```

#### 5.2 Tab Navigation
```jsx
// frontend/src/App.jsx
import { useState } from 'react';
import Home from './pages/Home';
import Wellness from './pages/Wellness';
import Finance from './pages/Finance';
import Content from './pages/Content';
import Relationships from './pages/Relationships';
import Critic from './pages/Critic';

export default function App() {
  const [activeTab, setActiveTab] = useState('home');
  
  const screens = {
    home: <Home />,
    wellness: <Wellness />,
    finance: <Finance />,
    content: <Content />,
    relationships: <Relationships />,
    critic: <Critic />
  };
  
  return (
    <div className="app">
      <div className="content">
        {screens[activeTab]}
      </div>
      <div className="navbar">
        {Object.keys(screens).map(tab => (
          <button
            key={tab}
            onClick={() => setActiveTab(tab)}
            className={activeTab === tab ? 'active' : ''}
          >
            {tab.charAt(0).toUpperCase() + tab.slice(1)}
          </button>
        ))}
      </div>
    </div>
  );
}
```

---

## Phase 6: Memory & Embeddings (Week 6)

### Goal
Add semantic search so agents learn from past sessions.

### What to Build

```python
# backend/memory/embeddings.py
from anthropic import Anthropic

class EmbeddingsManager:
    def __init__(self, client: Anthropic):
        self.client = client
    
    async def embed(self, text: str) -> List[float]:
        """Generate embedding using Claude"""
        # Note: Claude doesn't expose embeddings API, so use Ollama or another service
        # For now, use a simpler approach: store full text for semantic search
        pass

# Or use an open-source embeddings model
import sentence_transformers

class EmbeddingsManager:
    def __init__(self):
        self.model = sentence_transformers.SentenceTransformer('all-MiniLM-L6-v2')
    
    def embed(self, text: str) -> List[float]:
        return self.model.encode(text).tolist()
```

---

## Full Development Timeline

| Week | Phase | Deliverable |
|---|---|---|
| 1 | Foundation | Backend can receive requests, Orchestrator routes, Agents exist |
| 2 | Real Data | Agents fetch from DB/APIs, HealthKit sync works |
| 3 | Inter-Agent Communication | Critic queries other agents, LLM integration |
| 4 | Daily Briefing | All agents work together, daily schedule fires |
| 5 | Frontend | Beautiful home screen, tab navigation |
| 6 | Memory/Embeddings | Semantic search, agent learns from past |
| 7 | Polish | Error handling, logging, optimizations |
| 8 | Deploy | Backend on cloud, frontend hosted |
| 9 | iOS (optional) | Native iOS app if desired |
| 10 | App Store | Launch on App Store |

---

## Key Commands During Development

```bash
# Backend

# Start dev server
python backend/main.py

# Run migrations
alembic upgrade head

# Test an endpoint
curl -H "Authorization: Bearer <token>" http://localhost:8000/v1/briefing/today

# Check database
psql -U postgres -d jarvis
SELECT * FROM user_events LIMIT 10;

# Frontend

# Start dev server
npm start

# Build for production
npm run build

# Deploy to Vercel
vercel
```

---

## Troubleshooting Checklist

**API returns 401 Unauthorized**
- [ ] JWT token in Authorization header
- [ ] Token not expired
- [ ] Secret key matches

**Agent doesn't respond to query**
- [ ] Agent registered with orchestrator
- [ ] Agent implements handle_query()
- [ ] Query type matches implementation

**Database connection fails**
- [ ] PostgreSQL running locally or cloud URL correct
- [ ] DATABASE_URL env var set
- [ ] Database exists

**Frontend shows cached data instead of fresh**
- [ ] Check localStorage/IndexedDB
- [ ] Clear cache: localStorage.clear()
- [ ] Network tab shows API call failed

**Cron job doesn't fire**
- [ ] APScheduler running
- [ ] System timezone matches schedule
- [ ] Check logs for errors

