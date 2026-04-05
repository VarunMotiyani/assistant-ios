# JARVIS — System Architecture

## Overview

JARVIS is a three-layer system:

```
Layer 1 — iOS Native App (SwiftUI)
  Handles: UI, permissions, HealthKit, notifications, widgets, App Intents/Siri, offline cache

Layer 2 — Backend Orchestration (FastAPI + Python)
  Handles: agent orchestration (LangGraph), memory, integrations, scheduling, AI reasoning, rules engine

Layer 3 — Data & AI Services
  PostgreSQL (persistent store)
  pgvector (semantic memory)
  Redis (cache, ephemeral state)
  LLM API (Anthropic Claude)
  External: Google Calendar, Notion, Finance aggregator, HealthKit sync, News feeds
```

***

## iOS App Architecture

**Pattern:** MVVM + Clean Architecture with SwiftUI
**Concurrency:** Swift Concurrency (async/await, actors)
**State management:** @Observable macro (iOS 17+) + @EnvironmentObject for global app state
**Persistence (local):** SwiftData for offline cache, UserDefaults for preferences, Keychain for secrets
**Networking:** URLSession wrapped in a custom NetworkClient actor

### iOS Core Managers

**NotificationManager**
- Schedules local notifications (reminders, check-ins, hydration, motivational nudges)
- Handles incoming remote push (APNs) from backend
- Routes notification tap actions to the correct agent view
- Requests UNAuthorizationOptions: [.alert, .badge, .sound, .criticalAlert]

**HealthKitManager**
- Permission request flow — granular per data type, user-explained
- Sleep: HKCategoryTypeIdentifier.sleepAnalysis (in-bed, asleep, REM, deep, core stages)
- Workouts: HKWorkoutType (duration, calories, type, heart rate during workout)
- Heart rate, HRV, resting heart rate: HKQuantityType
- Steps, active energy, stand hours: HKQuantityType
- Body weight, body fat: HKQuantityType (manual or Apple Watch)
- Background delivery via HKObserverQuery for Apple Watch data sync
- Posts health snapshots to backend every 30 minutes via BGAppRefreshTask

**BackgroundTaskManager**
- BGAppRefreshTask identifier: "com.jarvis.app.refresh" — periodic digest (every 15 min)
- BGProcessingTask identifier: "com.jarvis.app.nightly" — heavy sync (nightly, requires charging + WiFi)
- On refresh: fetch latest briefing from backend, update local SwiftData cache, schedule any new local notifications
- On nightly: full HealthKit sync, Notion sync, finance sync

**NetworkClient (actor)**
- Base URL from environment config
- JWT Bearer token injected from Keychain on every request
- Retry logic: 3 retries with exponential backoff (1s, 2s, 4s)
- WebSocket connection to /ws/nudges for real-time motivational pushes
- Offline mutation queue: stores failed POST/PATCH operations in SwiftData, replays on reconnect

### iOS Views (Tab Structure)

```
TabBar — 5 tabs
├── Home (house icon) — Today Command Center
│   ├── DailyBriefingCard
│   ├── WellnessSnapshotCard
│   ├── FinanceSnapshotCard
│   ├── TopNewsCard (3 AI/tech headlines)
│   ├── PeopleToContactCard
│   └── CriticInsightCard
├── Admin (checklist icon) — Life Admin Agent
│   ├── InboxView
│   ├── CalendarSyncView
│   ├── TaskDetailView
│   └── AddTaskSheet
├── Wellness (heart icon) — Health & Fitness Agent
│   ├── SleepTrendView (7/30-day chart)
│   ├── WorkoutLogView
│   ├── MoodCheckInView
│   ├── HydrationTrackerView
│   ├── MeetingLoadView
│   └── SupplementTrackerView
├── Finance (chart icon) — Finance Agent
│   ├── NetWorthView
│   ├── HoldingsView
│   ├── CashflowView
│   ├── AIAdviceView
│   └── MarketPulseView
└── Brain (brain icon) — Content + Critic + People
    ├── TodayFeedView
    ├── SavedCardsView
    ├── CriticWorkspaceView
    └── PeopleView (Relationships Agent)
```

### App Intents (Siri / Shortcuts)

| Intent | Trigger phrase | Action |
|---|---|---|
| LogMoodIntent | "Log my mood" | Opens mood check-in sheet |
| TodayBriefingIntent | "What's my day look like" | Opens Today screen, reads briefing aloud |
| AskCriticIntent | "Ask Jarvis to critique my idea" | Opens Critic workspace |
| LogWorkoutIntent | "Log my workout" | Opens workout quick-log |
| ShowPeopleIntent | "Who should I reach out to" | Shows reach-out list |
| FinanceCheckIntent | "How's my portfolio" | Reads portfolio delta aloud |

***

## Backend Architecture

**Framework:** FastAPI (Python 3.12)
**Orchestration:** LangGraph multi-agent graph
**LLM:** Claude claude-opus-4 for deep reasoning, Claude claude-haiku-3 for fast summaries and nudges
**Task queue:** Celery + Redis broker
**Scheduler:** Celery Beat for cron-style jobs

### Backend Module Map

```
backend/
├── main.py                          # FastAPI app, routers, CORS, middleware
├── config.py                        # Pydantic-settings, all env vars
├── auth/
│   ├── jwt_handler.py
│   └── middleware.py
├── agents/
│   ├── base_agent.py                # BaseAgent: shared tools, memory access, LLM client
│   ├── life_admin.py
│   ├── wellness.py
│   ├── finance.py
│   ├── content.py
│   ├── relationships.py
│   └── critic.py
├── orchestrator/
│   └── graph.py                     # LangGraph StateGraph definition
├── integrations/
│   ├── google_calendar.py
│   ├── notion.py
│   ├── healthkit_sync.py
│   ├── finance_aggregator.py
│   └── news_feeds.py
├── memory/
│   ├── models.py                    # SQLAlchemy ORM models
│   ├── postgres_store.py            # CRUD operations
│   └── vector_store.py             # pgvector semantic search
├── scheduler/
│   └── tasks.py                     # Celery tasks: briefing, nightly sync, weekly review
├── push/
│   └── apns_sender.py               # APNs HTTP/2 push sender
└── routers/
    ├── agents.py
    ├── health.py
    ├── finance.py
    ├── content.py
    ├── people.py
    └── notifications.py
```

***

## Daily Briefing Data Flow

```
6:45 AM — Celery Beat fires generate_daily_briefing task

Step 1 — Parallel data fetch
  ├── Google Calendar API → events for next 24 hours
  ├── Notion Tasks DB → overdue + due today
  ├── PostgreSQL health_snapshots → last sleep record
  ├── PostgreSQL holdings → portfolio value yesterday vs today
  ├── News feeds → top 5 AI/tech/startup stories (last 24h)
  └── PostgreSQL contacts → people due for contact today

Step 2 — LangGraph orchestrator
  ├── LifeAdminAgent node → task priorities, critical warnings, bill alerts
  ├── WellnessAgent node → recovery score, workout recommendation
  ├── FinanceAgent node → portfolio delta, financial flag, savings rate alert
  ├── ContentAgent node → ranked top 3 stories with "why it matters"
  └── RelationshipAgent node → who to contact today + suggested message type

Step 3 — Merge + persist
  └── DailyBriefing object written to PostgreSQL (briefings table)

Step 4 — Push to device
  └── APNs silent push → iOS app wakes
          └── BGAppRefreshTask fetches briefing
                  └── SwiftData cache updated
                          └── Today screen renders with fresh data
```

***

## Security Model

| Concern | Implementation |
|---|---|
| iOS → Backend auth | JWT Bearer token, stored in iOS Keychain |
| Secrets on device | iOS Keychain only — no UserDefaults, no plist |
| HealthKit data | Encrypted in transit (TLS 1.3), user-permissioned per type |
| Finance credentials | Plaid access tokens stored server-side only |
| Notion OAuth | Server-side, encrypted at rest in PostgreSQL |
| APNs device tokens | Per-user in PostgreSQL, rotated on app launch |
| LLM prompts | No PII sent to LLM without explicit user consent toggle |

***

## Tech Stack Summary

| Component | Technology |
|---|---|
| iOS App | SwiftUI, Swift 6, SwiftData, HealthKit, UserNotifications, BGTaskScheduler, App Intents, WatchConnectivity |
| Backend | Python 3.12, FastAPI, LangGraph, LangChain, Celery, APScheduler |
| LLM | Anthropic Claude claude-opus-4 (reasoning), claude-haiku-3 (fast ops) |
| Database | PostgreSQL 16 + pgvector extension |
| Cache / Queue | Redis 7 |
| Push | Apple Push Notification service (APNs HTTP/2) |
| Finance | Plaid Investments API (US) or Smallcase/Kite Connect API (India) |
| Calendar | Google Calendar API v3 |
| Notes/Tasks | Notion API v1 |
| News | RSS feeds + Hacker News Algolia API + arXiv API + NewsAPI.org |
| Dev infra | Docker Compose |
| Prod infra | AWS ECS Fargate + RDS PostgreSQL + ElastiCache Redis |