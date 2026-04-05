# JARVIS — Agent Specifications

Each agent is a LangGraph node in the backend orchestration graph AND a corresponding iOS module that renders its output. This file specifies each agent's purpose, inputs, outputs, decision logic, notification behavior, and iOS UI responsibilities.

***

## Agent 1 — Life Admin Agent

**Purpose:** Eliminate all open loops. Bills, renewals, subscriptions, deadlines, assignments, calendar prep. Acts as a proactive executive assistant that notices what's pending and surfaces it before it becomes a crisis.

### Inputs
- Google Calendar events (next 7 days, fetched via OAuth)
- Notion "Tasks" database (all items with status != Done)
- Notion "Bills & Renewals" database (due date, amount, recurrence)
- User-captured quick tasks (via iOS AddTaskSheet or Siri "add task" intent)

### Outputs
- `DailyInbox` object: prioritized list of tasks due today or overdue
- `UpcomingWarnings` list: items due in next 3 days
- `CriticalAlerts` list: overdue items, missed payments, expired subscriptions
- Synced task updates back to Notion (status, due date edits)
- Calendar event prep notes (for meetings tomorrow: who, what, prep needed)

### Decision Logic
1. Fetch all tasks from Notion. Classify by: overdue / due today / due this week / no date.
2. Fetch calendar events. For any meeting tomorrow or today: check if there's a linked task or note. If not, flag "no prep note."
3. Fetch bills. If due within 5 days and status != paid: add to CriticalAlerts.
4. Ask LLM (claude-haiku-3): "Given these tasks and events, what are the 3 most important things this user must do today? Be specific."
5. Output prioritized DailyInbox with LLM-ranked top 3 highlighted.

### Notification Triggers
- 7:00 AM: "Good morning. You have [N] tasks due today. [Top task] is your most critical. Tap to see your inbox."
- 9:00 AM (if critical alert): "⚠️ [Bill name] is due in 2 days. Don't forget."
- 2:00 PM (mid-day check): "Quick check — you have [N] open tasks. How's progress?"
- 8:00 PM (evening): "End-of-day review. [X] tasks completed, [Y] still open. Reschedule tomorrow?"
- Ad-hoc: If a calendar event is added that has no prep task within 12 hours of the event → nudge.

### iOS Views
- **InboxView:** Card-style task list. Swipe right to complete, swipe left to reschedule. Priority badges (🔴 critical, 🟡 today, ⚪ this week).
- **CalendarSyncView:** Timeline view of next 7 days from Google Calendar. Tapping an event shows prep notes and linked tasks.
- **AddTaskSheet:** Quick capture: title, due date picker, category (work/personal/finance/health), urgency toggle.
- **BillsRenewalsView:** List with due date countdown badges. Mark as paid action.

### Notion Databases Required
```
Tasks DB:
  - Name (title)
  - Status (Not started / In progress / Done)
  - Due Date (date)
  - Category (select: Work, Personal, Finance, Health, Learning, Startup)
  - Priority (select: Critical, High, Medium, Low)
  - Notes (rich text)

Bills & Renewals DB:
  - Name (title)
  - Amount (number, INR)
  - Due Date (date)
  - Recurrence (select: Monthly, Quarterly, Annual, One-time)
  - Status (select: Unpaid, Paid, Auto-debit)
  - Category (select: Subscription, Utility, Insurance, EMI, Other)
```

***

## Agent 2 — Wellness Manager Agent

**Purpose:** Keep the user alive and performing. Track sleep, mood, workouts, hydration, supplement intake, and meeting overload. Detect patterns that predict burnout or underperformance and intervene before it's too late. Acts like a personal coach + concerned mom hybrid.

### Inputs
- HealthKit (via iOS sync): sleep stages, duration, efficiency, HRV, resting heart rate
- HealthKit: workouts (type, duration, calories, heart rate)
- HealthKit: steps, active energy, stand hours
- HealthKit: body weight (if tracked)
- Calendar: meeting count and total duration per day (Google Calendar)
- User mood check-in (1–10 scale + optional note, via iOS or notification tap)
- User hydration log (manual or Apple Watch water tracking)
- User supplement log (creatine 3–5g daily, whey 2–3 scoops — tracked manually in app)

### Outputs
- `RecoveryScore` (0–100): composite of sleep quality + HRV + resting HR + mood
- `WorkloadScore` (0–100): meeting hours + calendar density + task volume
- `DailyCoachingNote`: 2–3 sentence personalized recommendation
- `WorkoutRecommendation`: based on recovery score (light / moderate / heavy / rest)
- `SupplementReminder`: creatine and whey timing reminders
- Weekly wellness report: trends, streaks, anomalies

### Decision Logic
1. Pull last night's sleep from HealthKit. Score sleep: < 6h = poor (20pts), 6–7h = fair (50pts), 7–8h = good (75pts), 8h+ = excellent (100pts). Adjust for HRV if available.
2. Pull today's calendar. Count meeting minutes. If > 4 hours: high workload flag.
3. Pull yesterday's workout. If no workout in 3 days and recovery score > 60: flag "you should train today."
4. If recovery score < 40 AND workload > 70: send burnout warning.
5. Ask LLM (claude-haiku-3): "Recovery score [X], workload [Y], sleep [Z]h, mood [M]/10. Give a brutally honest 2-sentence coaching note for a bodybuilder/engineer who needs to perform today."
6. Check supplement log. If creatine not logged by 10 AM: send reminder. If whey not logged post-workout: nudge.

### Notification Triggers
- 7:30 AM: Morning mood check-in. "How are you feeling today? 1–10" (interactive notification with number picker).
- 8:30 AM: "Recovery score: [X]/100. [Coaching note]. Today's recommendation: [workout type]."
- 10:00 AM (if creatine not logged): "🏋️ Don't forget your creatine."
- Post-workout (if HealthKit detects workout ended): "Log your whey protein? Tap yes."
- 2:00 PM (hydration): "💧 Drink some water. You've been on calls all day."
- 9:00 PM (sleep prep): "Tomorrow is [day with X meetings]. Get to bed by [target time] for 8 hours."
- Burnout warning (ad-hoc): "⚠️ You've had heavy workload for 4 days and sleep under 6h for 3 nights. You need to rest today."

### iOS Views
- **WellnessDashboard (home widget):** Recovery score ring, today's sleep bar, steps ring.
- **SleepTrendView:** 14-day chart (bar chart). Tap any bar for that night's detail (stages, HRV, resting HR).
- **WorkoutLogView:** HealthKit workouts list. Filter by type. Tap for detailed stats.
- **MoodTrackerView:** Calendar heatmap + monthly average trend line.
- **SupplementTrackerView:** Daily checklist (creatine ✓, whey post-workout ✓, whey second scoop ✓). Streak counter.
- **HydrationTrackerView:** Daily water intake log with target line.
- **MeetingLoadView:** Today and tomorrow's call hours visualized as a timeline block view.

### Watch Complications
- Recovery score ring (circular complication)
- Next hydration nudge timer (small rectangular)
- Supplement streak (small inline)

***

## Agent 3 — Finance Manager Agent

**Purpose:** Complete financial awareness without manual tracking. Track portfolio value, investments, savings rate, cashflow, and market pulse. Give brutally honest financial coaching based on actual numbers. Flag anomalies and opportunities.

### Inputs
- Finance aggregator API (Plaid for US accounts; Smallcase API or Kite Connect for Indian equities/MF; manual CSV upload fallback)
- Portfolio snapshots (holdings, quantities, current prices)
- Bank transactions (optional: Plaid bank API or manual upload)
- Gold prices (public API: RBI or MCX)
- Market indices: Nifty50, Sensex, Nasdaq, S&P500 (free APIs: Yahoo Finance, NSE API)
- User-defined financial rules (savings rate target, emergency fund target, monthly budget)

### Outputs
- `NetWorthSnapshot`: total portfolio + savings + liquid cash
- `PortfolioDelta`: today's change (INR + percentage)
- `CashflowSummary`: monthly inflow vs outflow, top spending categories
- `SavingsRate`: monthly savings as percentage of income
- `FinancialAdvice`: LLM-generated coaching based on user's numbers and goals
- `MarketPulse`: 4–5 bullet market context (indices, sector moves, portfolio correlation)
- Weekly finance report: net worth trend, savings rate trend, portfolio performance

### Decision Logic
1. Fetch holdings from aggregator. Compute total portfolio value. Compare to yesterday.
2. If portfolio drops > 2% in a day: send market alert.
3. Fetch monthly transactions. Compute: income, expenses by category, savings. Calculate savings rate.
4. If savings rate this month < user target: flag.
5. If a holding is down > 15% from cost basis: flag for review (not sell advice, just "review this").
6. Ask LLM (claude-opus-4): "User's portfolio: [data]. Monthly cashflow: [data]. Savings rate: [X]% vs target [Y]%. Give 3 specific, harsh, actionable financial insights. No fluff."
7. Compute emergency fund coverage = liquid savings / monthly expenses. If < 3 months: alert.

### Notification Triggers
- 9:30 AM (market open, weekdays): "Markets open. Portfolio: ₹[X] ([+/-Y]% today)."
- 4:00 PM (market close, weekdays): "Markets closed. [Top mover in portfolio] moved [X]%."
- Monthly (1st of month): "Monthly financial review ready. Savings rate: [X]%. Net worth: ₹[Y]. Tap to review."
- Ad-hoc alerts:
  - "Portfolio down 2.5% today. [Cause context if available]."
  - "You've spent ₹[X] on [category] this month, [Y]% above your average. Heads up."
  - "Emergency fund is [X] months of expenses. Recommended minimum is 3."

### iOS Views
- **NetWorthView:** Big number up top. Sparkline trend (30/90/365 day). Breakdown donut chart (equity, MF, FD, gold, cash).
- **HoldingsView:** List of all holdings. Each row: name, quantity, current price, P&L, P&L%. Color-coded.
- **CashflowView:** Monthly inflow/outflow bar chart. Category breakdown. Savings rate indicator.
- **AIAdviceView:** Scrollable cards of AI-generated insights. Each card: title, insight, data point, action.
- **MarketPulseView:** Nifty50, Sensex, Nasdaq indices. User's watchlist. Fear & Greed-style sentiment summary.

***

## Agent 4 — AI & Tech Content Curator Agent

**Purpose:** Be the user's personal research intern + editorial curator for AI, tech, startups, and market news. Curate, summarize, personalize, and store. Generate idea cards, talking points, and knowledge facts daily. Track what the user engages with to get smarter over time.

### Inputs
- RSS feeds: TechCrunch, The Verge, Ars Technica, VentureBeat, Wired
- Hacker News: top 10 stories daily (Algolia API)
- arXiv: new papers in cs.AI, cs.LG, cs.CL (filtered for relevance)
- ProductHunt: top 5 daily launches
- NewsAPI.org: "AI" + "startup" + "India tech" keywords
- User engagement history: which cards were saved, liked, or expanded (stored in PostgreSQL)
- User's Notion "Knowledge Cards" database (two-way sync)

### Outputs
- `DailyFeed`: 8–12 curated items, ranked by relevance to user profile
- `TopStories`: top 3 items selected for the Today home screen
- `IdeaCards`: for 1–2 items per day, a richer card with: summary, "why it matters for you," talking points, related concepts
- `KnowledgeFacts`: 1 random interesting AI/tech fact each day
- Notion sync: saved/starred cards pushed to user's Notion "Knowledge Cards" DB
- Weekly digest: top 10 stories of the week, top saved topic tags

### Decision Logic
1. Fetch all sources every 6 hours. Deduplicate by URL + title similarity.
2. For each item: generate 2-sentence summary and relevance score (1–10) using claude-haiku-3.
3. Relevance scoring prompt: "User is a Python backend engineer interested in: LLM deployment, AI agents, cloud infra (AWS/GCP), startups, Indian tech market, intraday trading. Score this article 1–10 for relevance."
4. Keep top 15 items. Further rank by: freshness + relevance + diversity of topics.
5. For top 2 items: generate full IdeaCard (claude-opus-4):
   - 5-bullet summary
   - "Why this matters for a software engineer/founder"
   - 3 talking points (for conversations or content creation)
   - Related concepts to explore
6. Track user engagement (tap, save, skip). Update relevance model weights in PostgreSQL user_preferences table.

### Notification Triggers
- 8:00 AM: "🧠 Today's top story: [Title]. [1-sentence hook]. Tap to read."
- 12:30 PM (lunch): "Quick AI update: [2nd top story title and hook]."
- 6:00 PM: "Your daily tech digest is ready. [X] stories curated for you."
- Weekly Sunday 10:00 AM: "Your week in tech. [Top topic] dominated this week. Weekly digest ready."
- Ad-hoc: If a major AI announcement breaks (high Hacker News score + trending): "🚨 Breaking: [title]"

### iOS Views
- **TodayFeedView:** Horizontal swipeable cards at top (top 3 stories). Below: full feed list. Each item: source icon, title, 2-line summary, relevance badge, save button.
- **IdeaCardView:** Full-screen card. Title, source, summary bullets, talking points, related concepts, save to Notion button.
- **SavedCardsView:** All starred/saved items. Searchable. Tags filter. Synced from Notion.
- **KnowledgeFactView:** Minimal card shown at bottom of Brain tab. One fact per day, animated reveal.
- **TopicsView:** Tag cloud of user's most-engaged topics. Tap to filter feed by topic.

### Notion Databases Required
```
Knowledge Cards DB:
  - Title (title)
  - Source URL (url)
  - Summary (rich text)
  - Talking Points (rich text)
  - Tags (multi-select: AI, LLM, Cloud, Startup, India, Market, etc.)
  - Saved Date (date)
  - User Notes (rich text — editable by user)
```

***

## Agent 5 — Relationship Manager Agent

**Purpose:** Prevent the user from being a bad friend, family member, or network contact due to a busy schedule and forgetful memory. Track important people, important dates, last contact, and relationship health. Prompt action at the right time with the right message.

### Inputs
- User-maintained "People" database in Notion (manually seeded, agent-assisted updates)
- Calendar events (detect when a person is mentioned in an event title or notes)
- User contact logs (when user marks "contacted" from the app)
- Birthday / anniversary dates (stored in Notion People DB)
- Optional: Gmail contact notes (if user connects Gmail)

### Outputs
- `TodayReachOuts`: 2–3 people to contact today (based on urgency score)
- `UpcomingMoments`: birthdays and anniversaries in next 14 days
- `RelationshipHealthScore` per contact: days since contact / frequency target
- `MessageDraft`: suggested opening message for each reach-out (LLM-generated, warm and personal)
- Weekly relationship digest: who you reached out to, who you missed, upcoming moments

### Decision Logic
1. For each contact: compute urgency score = (days since last contact / frequency target) * importance weight.
   - Frequency targets: Close friends/family = 14 days, Professional contacts = 30 days, Others = 60 days.
   - Importance weights: Close = 3, Professional = 2, Acquaintance = 1.
2. Sort by urgency score descending. Top 3 = TodayReachOuts.
3. Check birthdays/anniversaries in next 14 days. Add to UpcomingMoments.
4. For each TodayReachOut: generate message draft (claude-haiku-3) with context: name, relationship type, last known interaction topic, occasion if any. Tone: warm, casual, not AI-sounding.
5. If birthday is tomorrow and not yet acknowledged: escalate to critical notification.

### Notification Triggers
- 9:00 AM daily: "👥 Reach out today: [Name 1], [Name 2]. Tap to see suggested messages."
- 3 days before birthday/anniversary: "🎂 [Name]'s birthday is in 3 days. Have you planned something?"
- Day of birthday: "🎉 Today is [Name]'s birthday! Send a message now." with pre-drafted text ready to copy.
- If a contact hasn't been reached in 2x their target frequency: "⚠️ You haven't talked to [Name] in [X] days. They're probably missing you."

### iOS Views
- **PeopleView (main):** List of all contacts with relationship health ring. Color: green (on track), yellow (due soon), red (overdue).
- **UpcomingMomentsView:** Timeline card list. Each card: person's name, event type, days remaining, action button.
- **ReachOutListView:** Today's reach-out cards. Each card: person photo/initials, last contact date, suggested message draft, "Mark as contacted" button and "Copy message" button.
- **ContactDetailView:** Full profile. Photo, relationship type, birthday, last contact log, interaction history, notes, importance level.
- **AddContactSheet:** Name, relationship type, birthday, contact frequency preference, notes, tags.

### Notion Databases Required
```
People DB:
  - Name (title)
  - Relationship Type (select: Family, Close Friend, Friend, Professional, Acquaintance)
  - Birthday (date)
  - Anniversary / Important Date (date, optional)
  - Last Contacted (date)
  - Contact Frequency (select: Weekly, Biweekly, Monthly, Quarterly)
  - Importance (select: High, Medium, Low)
  - Notes (rich text)
  - Tags (multi-select)
```

***

## Agent 6 — Critic Agent

**Purpose:** Be the user's brutally honest, highly intelligent thinking partner. Tear apart startup ideas, validate plans, challenge assumptions, evaluate decisions with concrete reasoning. No motivational fluff. No false positivity. True signal only.

### Inputs
- User's explicit prompt (text or voice-to-text): idea, plan, decision, question
- Optional context auto-injected:
  - Life Admin: current task load and calendar density
  - Wellness: current recovery score and sleep trend (5 days)
  - Finance: net worth, savings rate, current runway
  - Content: recent saved articles (for market context)
- Semantic memory: user's past critic sessions (retrieved via pgvector similarity search)
- User-defined goals (stored in PostgreSQL user_goals table)

### Outputs
- `Verdict`: Red / Yellow / Green with one-line rationale
- `CoreCritique`: 3–5 specific, numbered problems or risks with the idea/plan
- `Assumptions`: what assumptions the user is making that may be wrong
- `MissingInformation`: what data or research is needed before proceeding
- `IfYouMustProceed`: 3 specific changes that would make the plan better
- `PersonalContext`: how this decision fits (or conflicts) with user's current life state
- Full critique stored in PostgreSQL for future reference

### Decision Logic (LangGraph sub-graph for Critic)

```
Step 1 — Clarify
  LLM (claude-opus-4): "Restate the user's goal in one sentence. What are they actually asking for feedback on?"

Step 2 — Context Pull
  Pull: calendar load, recovery score, finance snapshot, past related critiques (semantic search)

Step 3 — Assumption Extraction
  LLM: "List all explicit and implicit assumptions this idea/plan makes."

Step 4 — Multi-axis Evaluation
  Evaluate on:
  - Market: Is there a real problem? Is there a paying customer?
  - Technical: Can this be built with user's current skills + time?
  - Personal: Is this aligned with user's current energy, finances, time?
  - Risk: What's the worst-case downside?
  - Timing: Is now the right time?

Step 5 — Verdict + Critique
  LLM: Synthesize into structured output. Tone: a senior founder/investor who genuinely wants the user to succeed but refuses to lie.

Step 6 — Store
  Save full session to PostgreSQL (critic_sessions table) and embed summary in vector store.
```

### Prompt Persona (inject into every critic session)
```
You are a highly intelligent, brutally honest critic and advisor. You have the combined perspective of:
- A senior software engineer with 10+ years of experience
- A startup founder who has failed and succeeded
- A financial advisor who has seen people lose money on bad ideas
- A life coach who prioritizes long-term wellbeing over short-term excitement

Your job is NOT to encourage. Your job is to be RIGHT. Identify every flaw, every weak assumption, every risk. Then — and only then — tell the user what they'd need to do to make it work.

Do NOT:
- Use phrases like "Great idea!", "You've got this!", "That's exciting"
- Be vague ("this might be challenging")
- Give generic advice

DO:
- Be specific ("Your assumption that X is false because Y data shows Z")
- Give concrete actions ("Before proceeding, answer these 3 questions")
- Reference the user's actual life context (their recovery score is low, they're low on savings, they're already overloaded — factor this in)
```

### Notification Triggers
- No proactive notifications for Critic (demand-driven only)
- Exception: If the user's last critic session was more than 2 weeks ago and they have added 3+ "Startup" tasks in Life Admin: "💡 You've been building something. Want Jarvis to critique your plan?"

### iOS Views
- **CriticWorkspaceView:** Clean, minimal input area (text or voice). Submit button. Below: history of past sessions in accordion cards.
- **CriticResultView:** Full-screen result. Verdict badge (red/yellow/green) at top. Sections: Core Critique, Assumptions, Missing Info, If You Must Proceed, Personal Context. Share as text button.
- **CriticHistoryView:** Chronological list of past sessions. Each has: date, topic, verdict badge, short summary. Tap to expand full critique.