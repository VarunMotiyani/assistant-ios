# JARVIS Frontend Specification

## Overview

The frontend is **simple and dumb**. It just:
- Shows what the backend sends
- Collects user input
- Sends requests to backend
- Caches data locally for offline viewing

**The backend does all the thinking.**

---

## Frontend Options

You have flexibility here. You can build:

### Option A: Mobile App (Native iOS with SwiftUI)
- Native performance
- HealthKit integration (automatic sync of workouts, sleep, etc.)
- Push notifications via APNs
- Home screen widgets
- Siri shortcuts

### Option B: Web App (Browser - React/Vue)
- Works on all devices
- Simpler to build and deploy
- No native setup complexity
- Offline-first with IndexedDB

### Option C: Hybrid (Web + iOS)
- Backend serves same API to both
- Web for desktop/tablet
- iOS app for Apple Watch integration and HealthKit

**Recommendation for you:** Start with **Web (React or Vue)** for speed. iOS can come later.

---

## What the Frontend Does

### 1. Authentication
```
User enters email → Backend sends magic link → User clicks link → 
Token received → Stored locally → Included in all requests
```

### 2. Display Dashboard (Home Screen)
```
GET /briefing/today → Shows:
- Top 3 tasks (from LifeAdmin)
- Recovery score ring (from Wellness)
- Portfolio value + delta (from Finance)
- Top 3 stories (from Content)
- People to reach out to (from Relationships)
```

### 3. Display Agent Panels
```
User clicks on a panel → Frontend calls GET /agents/{agent}/query
→ Shows agent-specific dashboard:

Wellness panel:
  - Sleep chart (7-day)
  - Workout history
  - Mood log
  - Supplement checklist

Finance panel:
  - Holdings list
  - Net worth chart
  - Cashflow breakdown

LifeAdmin panel:
  - Task inbox
  - Calendar sync
  - Bills due

Content panel:
  - Feed items
  - Saved articles
  - Idea cards

Relationships panel:
  - People list
  - Reach-out suggestions
  - Upcoming birthdays
```

### 4. User Input → Backend
```
User fills out form:
  - New task: POST /tasks
  - Log mood: POST /health/mood
  - Mark task done: PATCH /tasks/{id}
  - Ask critic: POST /agents/critic/evaluate
  - Log supplement: POST /health/supplement
  - Save article: POST /content/interact
  - Contact someone: POST /contacts/{id}/log-interaction
```

### 5. Real-Time Updates (WebSocket)
```
Frontend connects to: WS /ws/updates
Server sends events:
  - "briefing_updated" → refresh home screen
  - "portfolio_alert" → show alert if portfolio drops 2%+
  - "notification" → show badge/banner
```

### 6. Offline-First Caching
```
On-app-open:
  - Load last briefing from local storage
  - Load last health snapshot from local storage
  - Load last portfolio value from local storage

When internet comes back:
  - Sync any pending actions (new task, mood log, etc.)
  - Refresh all data from backend
```

---

## Key Screens

### Screen 1: Home (Daily Briefing)
```
┌─────────────────────────────────┐
│ JARVIS Home                    │
├─────────────────────────────────┤
│                                   │
│ 📅 Today, Saturday April 5        │
│                                   │
│ ┌──────────────────────────────┐  │
│ │ 🏥 RECOVERY SCORE             │  │
│ │ 62/100 - Good                 │  │
│ │ "Ready for moderate work"     │  │
│ └──────────────────────────────┘  │
│                                   │
│ ┌──────────────────────────────┐  │
│ │ ✓ TOP 3 TASKS TODAY           │  │
│ │ 🔴 Close Q1 billing           │  │
│ │ 🟡 Review metrics             │  │
│ │ ⚪ Respond to 3 emails        │  │
│ └──────────────────────────────┘  │
│                                   │
│ ┌──────────────────────────────┐  │
│ │ 💰 PORTFOLIO                  │  │
│ │ ₹5,012,500  +₹12,500 (+1.2%)  │  │
│ └──────────────────────────────┘  │
│                                   │
│ ┌──────────────────────────────┐  │
│ │ 🧠 TOP NEWS TODAY             │  │
│ │ "OpenAI releases GPT-4.5..."  │  │
│ └──────────────────────────────┘  │
│                                   │
│ ┌──────────────────────────────┐  │
│ │ 👥 REACH OUT TODAY            │  │
│ │ Rahul - last talked 18d ago   │  │
│ │ Dad - birthday in 5d          │  │
│ └──────────────────────────────┘  │
│                                   │
└─────────────────────────────────┘

Swipe left → Finance details
Swipe right → Wellness details
Tap card → Open full panel
```

### Screen 2: Wellness Panel
```
┌─────────────────────────────────┐
│ Wellness                         │
├─────────────────────────────────┤
│ Recovery Score: 62/100          │
│ ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  │
│                                  │
│ Sleep Last Night                │
│ Duration: 7h 12m  ✓             │
│ Efficiency: 86%   ✓             │
│ Deep: 90m  REM: 110m            │
│                                  │
│ [Chart: 7-day sleep trend]       │
│                                  │
│ Mood Check-in                   │
│ How are you today? [1-10]        │
│ [Picker showing 1...10]          │
│                                  │
│ Workouts                         │
│ Apr 4: Strength - 60min - 400cal │
│ Apr 2: Running - 30min - 250cal  │
│ Apr 1: Strength - 50min - 380cal │
│                                  │
│ Supplement Tracker              │
│ ✓ Creatine (24-day streak)      │
│ ⏳ Whey protein (not logged)    │
│ [Log button]                     │
│                                  │
└─────────────────────────────────┘
```

### Screen 3: Critic Session
```
┌─────────────────────────────────┐
│ Critic                           │
├─────────────────────────────────┤
│ What should Jarvis critique?    │
│                                  │
│ [Text input area]                │
│ "Should I build an AI course?"  │
│                                  │
│ [Include my life context?] ✓    │
│ [Mic button] 🎤                 │
│                                  │
│ [SEND]                           │
│                                  │
│ ─────────────────────────────────│
│ PAST SESSIONS                    │
│                                  │
│ 🟡 "Build AI course?" - Apr 5   │
│ 🟢 "Pivot product?" - Apr 1     │
│ 🔴 "Full-time startup?" - Mar 28│
│                                  │
└─────────────────────────────────┘

On tap → Expand to full critique:
[Verdict banner]
[Core critique]
[Assumptions]
[Missing info]
[If you must proceed]
[Personal context]
```

### Screen 4: Relationships
```
┌─────────────────────────────────┐
│ Relationships                    │
├─────────────────────────────────┤
│ REACH OUT TODAY                 │
│                                  │
│ Rahul (Friend)                  │
│ Last contacted 18 days ago      │
│ "Hey Rahul! Been a while..."   │
│ [Copy message] [Mark contacted] │
│                                  │
│ ─────────────────────────────────│
│ UPCOMING MOMENTS                │
│                                  │
│ 🎂 Mom's birthday - in 5 days   │
│ 💍 Anniversary - in 12 days     │
│                                  │
│ ─────────────────────────────────│
│ ALL CONTACTS (sorted by urgency) │
│                                  │
│ 🔴 Rahul (Friend) - overdue     │
│ 🟡 Dad (Family) - due soon      │
│ ⚪ Boss (Work) - on track       │
│                                  │
└─────────────────────────────────┘
```

---

## API Calls per Screen

### Home Screen (on app open)
```
GET /briefing/today
├─ Life Admin data
├─ Wellness data
├─ Finance data
├─ Content top stories
└─ Relationship data

Cache in localStorage for offline
```

### Wellness Panel
```
GET /health/snapshots/latest      (today's data)
GET /health/snapshots/range?start=7d_ago  (chart data)
```

### Finance Panel
```
GET /finance/holdings
GET /finance/net-worth
GET /finance/cashflow
```

### Content Panel
```
GET /content/feed/today
GET /content/idea-card/{item_id}  (when user clicks a story)
```

### Relationships Panel
```
GET /contacts/reach-outs-today
GET /contacts/upcoming-moments
GET /contacts
```

### Task Actions
```
POST /health/mood              (logging mood)
POST /health/supplement        (logging supplement)
PATCH /tasks/{id}              (mark complete)
POST /contacts/{id}/log-interaction  (mark reached out)
```

### Critic
```
POST /agents/critic/evaluate   (submit for critique)
GET /agents/critic/sessions    (fetch history)
GET /agents/critic/sessions/{id}  (fetch specific session)
```

---

## Local Storage / Caching Strategy

### Web (IndexedDB)
```javascript
// On app open
const briefing = await db.briefings.get(today);
if (!briefing) {
  const fresh = await fetch('/briefing/today');
  await db.briefings.add(fresh);
}

// Show cached briefing while fetching fresh data
render(briefing);
fetch('/briefing/today').then(fresh => {
  if (different(briefing, fresh)) {
    updateUI(fresh);
    store(fresh);
  }
});
```

### iOS (SwiftData)
```swift
// Fetch latest from backend
let briefing = try await fetchBriefing()

// Save to SwiftData
try modelContext.insert(briefing)
try modelContext.save()

// Show immediately even if offline
@Query var cachedBriefing: [Briefing]
Text(cachedBriefing.first?.title ?? "Loading...")
```

---

## State Management

**Keep it simple.** Frontend state = UI state only.

```javascript
// What frontend manages:
{
  user: { id, email },
  uiState: { 
    activeTab: "home",
    isLoading: false,
    error: null
  },
  cache: {
    briefing: { date, data },
    health: { timestamp, data },
    portfolio: { timestamp, data }
  }
}

// What backend manages:
// Everything else (all the AI, memory, logic)
```

---

## Notifications

### Push Notifications (Backend → Frontend)

Backend sends via WebSocket or APNs (iOS):
```json
{
  "title": "Recovery score low",
  "body": "You've had 3 nights < 6h sleep. Rest recommended.",
  "action": "open://wellness",
  "priority": "high"
}
```

Frontend shows banner and stores in local notification center.

### Local Notifications (Frontend only)

Schedule time-based reminders:
```javascript
// Schedule mood check-in at 7:30 AM
scheduleNotification({
  title: "How are you feeling?",
  body: "Take 10 seconds to log your mood",
  trigger: { hour: 7, minute: 30 },
  action: "open://health/mood"
});
```

---

## Offline Mode

When internet is down:

1. **Show cached data** (last known briefing, portfolio, etc.)
2. **Disable actions** that need backend (ask critic, save to Notion, etc.)
3. **Queue pending actions** (new task, mood log, supplement log)
4. **On reconnect:** Sync queued actions to backend

Example:
```javascript
if (!navigator.onLine) {
  // Show cached briefing
  const cached = localStorage.getItem('briefing');
  render(JSON.parse(cached));
  
  // Queue action
  const pendingAction = {
    type: 'mood_logged',
    data: { score: 7 },
    timestamp: Date.now()
  };
  localStorage.setItem('pending_actions', JSON.stringify([pendingAction]));
}

// When online again
window.addEventListener('online', async () => {
  const pending = JSON.parse(localStorage.getItem('pending_actions'));
  for (const action of pending) {
    await fetch(actionEndpoint, { method: 'POST', body: action });
  }
  localStorage.removeItem('pending_actions');
});
```

---

## Building the Frontend

### Tech Stack Options

**Option 1: React (JavaScript)**
- Fastest to build
- Lots of libraries
- Requires Node.js/npm

```bash
npx create-react-app jarvis-frontend
cd jarvis-frontend
npm install axios zustand
```

**Option 2: Vue (JavaScript)**
- Similar to React but simpler
- Great docs

```bash
npm create vue@latest
cd jarvis-frontend
npm install axios pinia
```

**Option 3: Svelte (JavaScript)**
- Smallest bundle
- Compiles away framework overhead

**Option 4: SwiftUI (iOS only)**
- Native iOS app
- More complex but best UX
- Can access HealthKit

---

## Project Structure

### Web (React)
```
frontend/
├── public/
├── src/
│   ├── App.jsx
│   ├── index.css
│   ├── pages/
│   │   ├── Home.jsx
│   │   ├── Wellness.jsx
│   │   ├── Finance.jsx
│   │   ├── Content.jsx
│   │   ├── Relationships.jsx
│   │   └── Critic.jsx
│   ├── components/
│   │   ├── BriefingCard.jsx
│   │   ├── TaskItem.jsx
│   │   ├── HealthChart.jsx
│   │   └── ...
│   ├── api/
│   │   └── client.js      # Fetch wrapper
│   ├── store/
│   │   └── index.js       # Zustand store
│   ├── utils/
│   │   └── cache.js       # LocalStorage/IndexedDB
│   └── styles/
│       └── globals.css
├── package.json
└── vite.config.js
```

### iOS (SwiftUI)
```
ios/
├── Jarvis/
│   ├── App.swift
│   ├── Screens/
│   │   ├── HomeScreen.swift
│   │   ├── WellnessScreen.swift
│   │   ├── FinanceScreen.swift
│   │   ├── ContentScreen.swift
│   │   ├── RelationshipsScreen.swift
│   │   └── CriticScreen.swift
│   ├── Components/
│   │   ├── BriefingCard.swift
│   │   ├── TaskItem.swift
│   │   └── ...
│   ├── Services/
│   │   ├── APIClient.swift
│   │   ├── HealthKitManager.swift
│   │   └── StorageManager.swift
│   ├── Models/
│   │   ├── Briefing.swift
│   │   ├── Task.swift
│   │   ├── Contact.swift
│   │   └── ...
│   └── Views/
│       └── ContentView.swift
└── Jarvis.xcodeproj
```

---

## Key Implementation Notes

### 1. API Client Wrapper
Always use a wrapper so you don't repeat auth/error handling:

```javascript
// api/client.js
const apiClient = {
  async get(endpoint) {
    const token = localStorage.getItem('auth_token');
    const response = await fetch(`${API_BASE}/v1${endpoint}`, {
      headers: {
        'Authorization': `Bearer ${token}`,
        'Content-Type': 'application/json'
      }
    });
    if (response.status === 401) redirectToLogin();
    return response.json();
  },
  
  async post(endpoint, data) {
    // same with method: 'POST', body: JSON.stringify(data)
  }
};
```

### 2. Error Handling
Every API call needs error handling:

```javascript
try {
  const briefing = await fetch('/briefing/today');
} catch (error) {
  // Network error - show "offline" message
  showError("Can't reach backend. Showing cached data.");
  useCachedBriefing();
} catch (error) {
  if (error.status === 401) redirectToLogin();
  showError(error.message);
}
```

### 3. Loading States
Show spinners while fetching:

```javascript
const [loading, setLoading] = useState(false);

useEffect(() => {
  setLoading(true);
  fetch('/briefing/today')
    .then(data => setBriefing(data))
    .finally(() => setLoading(false));
}, []);

return loading ? <Spinner /> : <BriefingView briefing={briefing} />;
```

---

## Don't Build (Backend Does It)

❌ Don't implement:
- LLM API calls (backend does this)
- Agent orchestration (backend does this)
- Data persistence (backend does this)
- Integration syncing (backend does this)
- Complex business logic (backend does this)

✅ Do implement:
- Beautiful UI
- Responsive design
- Offline caching
- Form validation
- Charts/visualizations
- Smooth animations

