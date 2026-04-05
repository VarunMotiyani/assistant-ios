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
    actions: [1,2,3,4,5,6,7,8,9,10] Zerodha integration
- Holdings, positions, order history
- Requires Zerodha account + API subscription (₹2000/month)

Option C — **Manual CSV upload (fallback)**
- User exports from broker app and uploads CSV
- Backend parses standard formats (Zerodha, Groww, Upstox)

**For US accounts: Plaid Investments API**
- Plaid Link flow for US brokerage accounts
- Holdings, transactions, investment accounts

**Backend (finance_aggregator.py):**
```python
def get_holdings(user_id: str) -> list[Holding]:
    # Fetch from configured aggregator
    # Normalize to common Holding schema
    # Upsert to holdings table

def get_portfolio_value(user_id: str) -> PortfolioValue:
    # Return: total_value, day_change, day_change_pct

def get_transactions(user_id: str, since: date) -> list[Transaction]:
    # Fetch recent transactions
```

**Market data (free APIs):**
- NSE/BSE indices: `https://query1.finance.yahoo.com/v8/finance/chart/^NSEI`
- Individual stock quotes: Yahoo Finance API (yfinance Python library)
- Gold prices: MCX data or goldapi.io
- Mutual fund NAV: AMFI India public API (`https://api.mfapi.in/mf/{scheme_code}`)

---

## News Feeds

**Sources and fetch schedule (every 6 hours):**

| Source | Method | Filter |
|---|---|---|
| Hacker News | Algolia API: `http://hn.algolia.com/api/v1/search?tags=front_page` | Score > 100 |
| TechCrunch | RSS: `https://techcrunch.com/feed/` | All |
| The Verge | RSS: `https://www.theverge.com/rss/index.xml` | All |
| VentureBeat | RSS: `https://venturebeat.com/feed/` | AI category |
| arXiv | API: `http://export.arxiv.org/api/query?search_query=cat:cs.AI+OR+cat:cs.LG` | Last 24h, max 10 |
| ProductHunt | GraphQL API | Daily top 5 |
| NewsAPI | `https://newsapi.org/v2/everything?q=AI+startup` | Last 24h |
| IndiaAI | RSS or scrape | India-specific AI news |

**Backend (news_feeds.py):**
```python
def fetch_all_sources() -> list[RawFeedItem]:
    # Fetch all sources in parallel (asyncio.gather)
    # Deduplicate by URL + title similarity (fuzz matching)
    # Return list of raw items

def score_and_summarize(items: list[RawFeedItem], user_id: str) -> list[FeedItem]:
    # For each item: call claude-haiku-3 for 2-sentence summary + relevance score
    # Store to feed_items table
    # Return top 15 by relevance * freshness

def generate_idea_card(item: FeedItem) -> IdeaCard:
    # Call claude-opus-4
    # Return: 5-bullet summary, why-it-matters, 3 talking points, related concepts
```

---

## APNs (Push Notifications)

**Backend (apns_sender.py):**
```python
import httpx
import jwt as pyjwt

class APNsSender:
    def __init__(self):
        self.team_id = settings.APNS_TEAM_ID
        self.key_id = settings.APNS_KEY_ID
        self.private_key = settings.APNS_PRIVATE_KEY  # .p8 contents
        self.bundle_id = "com.yourname.jarvis"

    def send(self, device_token: str, title: str, body: str,
             category: str = None, data: dict = None,
             interruption_level: str = "active"):
        # Generate JWT auth token (10-min expiry)
        # POST to api.push.apple.com (prod) or api.sandbox.push.apple.com (dev)
        # apns-priority: 10 (immediate) or 5 (power-conserving)
        # apns-push-type: alert or background
```

