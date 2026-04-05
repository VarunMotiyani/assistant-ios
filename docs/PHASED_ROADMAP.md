# JARVIS — Notification Strategy

## Philosophy
Notifications are the heartbeat of JARVIS. They should feel like a knowledgeable friend tapping you on the shoulder at exactly the right time — not a spam machine. Every notification must pass this test: "Would I be annoyed to receive this, or grateful?"

Rules:
- Max 6 notifications per day in normal mode, max 10 in "intense mode" (user opt-in)
- Never send 2 notifications within 30 minutes unless it's a critical alert
- Respect Do Not Disturb / Focus Mode (use iOS interruptionLevel)
- Every notification must have a direct action (tap → do the thing, not tap → open app home)
- Learn from dismissal: if user dismisses same notification type 3x, ask to reduce frequency

***

## Notification Schedule (Default, Weekday)

| Time | Agent | Type | Content | Action |
|---|---|---|---|---|
| 7:00 AM | Life Admin | Morning Briefing | "Good morning. [N] tasks due today. Critical: [top task]." | Opens Today screen |
| 7:30 AM | Wellness | Mood Check-in | "How are you feeling? 1–10" | Interactive: inline number reply |
| 8:30 AM | Wellness | Recovery Report | "Recovery: [X]/100. Sleep: [Y]h. Today's training: [recommendation]." | Opens Wellness tab |
| 9:00 AM | Relationships | Reach Out | "Reach out today: [Name 1], [Name 2]. Suggested message ready." | Opens Reach Out list |
| 9:30 AM | Finance | Market Open | "Markets open. Portfolio: ₹[X] ([+/-Y]% today)." | Opens Finance tab |
| 12:30 PM | Content | Lunch Update | "Quick read: [top story title]. [1-line hook]." | Opens story card |
| 2:00 PM | Wellness | Hydration | "💧 You've been in meetings all day. Drink water." | Opens hydration log |
| 4:00 PM | Finance | Market Close | "Markets closed. [Top portfolio mover] moved [X]%." | Opens holdings |
| 6:00 PM | Content | Daily Digest | "Your daily tech digest: [X] stories curated." | Opens Today Feed |
| 8:00 PM | Life Admin | Evening Review | "[X] tasks done. [Y] still open. Reschedule?" | Opens inbox |
| 9:00 PM | Wellness | Sleep Prep | "Tomorrow: [N] meetings. Target bedtime: [time] for 8h sleep." | Opens sleep view |

Weekend schedule: fewer notifications, no market open/close, relaxed briefing at 9 AM.

***

## Ad-Hoc / Triggered Notifications

| Trigger | Agent | Message | interruptionLevel |
|---|---|---|---|
| Portfolio drops > 2% in a day | Finance | "⚠️ Portfolio down [X]%. [Context]." | timeSensitive |
| Birthday tomorrow | Relationships | "🎂 [Name]'s birthday is tomorrow. Have you planned something?" | active |
| Birthday today | Relationships | "🎉 Today is [Name]'s birthday! Message ready to copy." | timeSensitive |
| Bill due in 2 days | Life Admin | "⚠️ [Bill] due in 2 days. ₹[amount]." | active |
| Burnout detected | Wellness | "⚠️ 4 heavy days + poor sleep. You need to rest today." | timeSensitive |
| Supplement not logged by 10 AM | Wellness | "🏋️ Don't forget your creatine." | passive |
| Post-workout (HealthKit) | Wellness | "Log your whey protein? Tap yes." | active |
| Breaking AI news (HN score > 500) | Content | "🚨 Breaking: [title]" | active |
| No critic session in 14 days + startup tasks added | Critic | "💡 Want Jarvis to critique your plan?" | passive |
| Contact overdue 2x frequency | Relationships | "⚠️ You haven't talked to [Name] in [X] days." | passive |

***

## iOS Implementation

### Permission Request Flow
```swift
UNUserNotificationCenter.current().requestAuthorization(
    options: [.alert, .badge, .sound, .criticalAlert]
) { granted, error in
    // handle
}
```

### Interrupt Levels
- `.passive` — delivered silently to notification center, no sound/banner
- `.active` — standard banner + sound
- `.timeSensitive` — breaks through Focus Mode (requires entitlement)
- `.critical` — breaks through silent mode (reserved for true emergencies)

### Interactive Notification Categories

**Mood Check-in:**
```swift
UNNotificationCategory(
    identifier: "MOOD_CHECKIN",
    actions: [1,2,3,4,5,6,7,8,9,10]ts, AI financial coaching.

### Backend
- [ ] holdings, transactions, net_worth_snapshots tables
- [ ] Finance aggregator integration (Smallcase or Kite Connect for INR; CSV fallback)
- [ ] Market data: Yahoo Finance / yfinance for NSE stocks + indices + gold + MF NAV
- [ ] Finance Agent (finance.py): portfolio delta, cashflow, savings rate, LLM advice
- [ ] All finance endpoints: /finance/holdings, /cashflow, /advice, /net-worth/history
- [ ] Celery tasks: 9:30 AM market open alert, 4:00 PM market close alert
- [ ] Finance summary included in /briefing/today

### iOS
- [ ] Settings: broker connect flow (Smallcase/Kite Connect OAuth or CSV upload)
- [ ] FinanceTab: NetWorthView (big number + sparkline), HoldingsView, CashflowView
- [ ] AIAdviceView: LLM insight cards
- [ ] MarketPulseView: indices, watchlist
- [ ] FinanceSnapshotCard on Home: portfolio value + day change

### Deliverable
Portfolio visible, P&L updating, market open/close notifications firing, AI advice cards generated.

---

## Phase 5 — Content Curator (Week 6)
**Goal:** Daily AI/tech feed curated and personalized, saved to Notion.

### Backend
- [ ] feed_items, user_feed_interactions tables
- [ ] News feeds integration (all sources, every 6 hours)
- [ ] Content Agent (content.py): fetch → deduplicate → score → summarize
- [ ] IdeaCard generation for top 2 items (claude-opus-4)
- [ ] User engagement tracking: POST /content/interact updates weights in user_preferences
- [ ] Notion sync: saved cards pushed to Knowledge Cards DB
- [ ] Celery task: 6-hour feed refresh + 8 AM and 6 PM push notifications

### iOS
- [ ] BrainTab: TodayFeedView (swipeable top cards + list), SavedCardsView
- [ ] IdeaCardView: full-screen rich card
- [ ] Content feed interaction: save, like, skip
- [ ] TopNewsCard on Home: top 3 stories
- [ ] KnowledgeFact: daily fact card at bottom of Brain tab

### Deliverable
Fresh AI/tech feed every 6 hours. Stories summarized with relevance scores. Top 2 get full IdeaCards. Saves sync to Notion.

---

## Phase 6 — Relationships (Week 7)
**Goal:** People CRM working, birthday/anniversary reminders, reach-out list.

### Backend
- [ ] contacts, contact_interactions tables
- [ ] Notion People DB sync (two-way)
- [ ] Relationship Agent (relationships.py): urgency score computation, reach-out selection, message drafts
- [ ] All /contacts endpoints
- [ ] Celery task: 9 AM reach-out notification, birthday/anniversary alerts

### iOS
- [ ] PeopleTab (inside BrainTab or standalone 5th tab): ContactsView, UpcomingMomentsView, ReachOutListView
- [ ] ContactDetailView: full profile, interaction history
- [ ] AddContactSheet
- [ ] ReachOutListView: message draft + "Mark contacted" + "Copy message" buttons
- [ ] PeopleToContactCard on Home

### Deliverable
People CRM populated from Notion. Daily reach-out nudges with drafted messages. Birthday alerts 3 days before.

---

## Phase 7 — Critic Agent (Week 8)
**Goal:** Brutally honest critic accessible via text/voice, with history and context injection.

### Backend
- [ ] critic_sessions table with pgvector embeddings
- [ ] Critic Agent (critic.py): full LangGraph sub-graph (clarify → context pull → assumptions → multi-axis eval → verdict)
- [ ] POST /critic/evaluate, GET /critic/sessions, GET /critic/sessions/{id}
- [ ] Semantic search on past sessions for context injection (pgvector)

### iOS
- [ ] CriticWorkspaceView: clean text input + voice (Speech framework)
- [ ] CriticResultView: verdict badge + all sections
- [ ] CriticHistoryView: past sessions list
- [ ] CriticInsightCard on Home: last critic verdict summary
- [ ] AskCriticIntent (App Intent for Siri)

### Deliverable
Type or speak an idea → get a brutal, structured critique with context from your real life data. History searchable. Accessible via Siri.

---

## Phase 8 — Polish + App Intents + Widgets (Week 9)
**Goal:** The Jarvis feel. Always-on presence via widgets, Lock Screen, Siri, and seamless UX.

### iOS
- [ ] Home screen widget (WidgetKit): Today snapshot — recovery score, top task, portfolio delta
- [ ] Lock screen widget: recovery score ring + top task
- [ ] All 6 App Intents registered and working with Siri
- [ ] Siri Shortcuts catalog: users can add all intents to Shortcuts
- [ ] App icon (dark + light + tinted variants)
- [ ] Haptic feedback throughout (on complete task, on critic verdict, on supplement log)
- [ ] Smooth transitions and animations (SwiftUI matchedGeometryEffect where appropriate)
- [ ] Onboarding wizard: 5-step setup guide (connect Notion → connect GCal → connect finance → HealthKit → choose notification schedule)
- [ ] Settings screen: complete — notification timing, supplement targets, savings goals, privacy controls, connected services

### Backend
- [ ] Rate limiting on all endpoints (Redis-based)
- [ ] Structured logging (structlog)
- [ ] Health check endpoint: GET /health
- [ ] API versioning in place

### Deliverable
Full Jarvis experience. Widgets on home/lock screen. Siri commands working. Onboarding guides new user setup. App feels native, polished, and fast.

---

## Phase 9 — Production Deploy (Week 10)
- [ ] AWS ECS Fargate for backend
- [ ] AWS RDS PostgreSQL (Multi-AZ for prod)
- [ ] AWS ElastiCache Redis
- [ ] GitHub Actions CI/CD pipeline
- [ ] TestFlight distribution
- [ ] Crashlytics or Sentry for crash reporting
- [ ] App Store submission prep

---

## Environment Variables Required

```bash
# Backend
DATABASE_URL=postgresql://user:pass@host:5432/jarvis
REDIS_URL=redis://host:6379
JWT_SECRET_KEY=
JWT_ALGORITHM=HS256

# LLM
ANTHROPIC_API_KEY=

# Google Calendar
GOOGLE_CLIENT_ID=
GOOGLE_CLIENT_SECRET=

# Notion
NOTION_CLIENT_ID=
NOTION_CLIENT_SECRET=

# Finance
SMALLCASE_API_KEY=
KITE_CONNECT_API_KEY=
KITE_CONNECT_SECRET=
PLAID_CLIENT_ID=
PLAID_SECRET=

# News
NEWSAPI_KEY=

# APNs
APNS_TEAM_ID=
APNS_KEY_ID=
APNS_PRIVATE_KEY=  # contents of .p8 file
APNS_BUNDLE_ID=com.yourname.jarvis
APNS_PRODUCTION=false

# iOS App (Info.plist / xcconfig)
API_BASE_URL=https://api.jarvis.yourdomain.com/v1
```

