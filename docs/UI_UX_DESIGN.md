# JARVIS UI/UX Design
## The Operating System for Your Life (Donna AI Edition)

**Vision:** A beautiful, action-driven operating system that manages your life like Donna from Suits — proactive, intelligent, and always one step ahead.

---

## 1. Core Philosophy

### Not Chat, Not Dashboard, But OS

```
❌ ChatGPT:  "Ask it questions" (reactive)
❌ Dashboard: "Shows data" (passive)
✅ JARVIS:   "Manages your life" (proactive)
           + Beautiful UI
           + Actionable buttons
           + Learns from your behavior
```

Every screen should answer: **"What does JARVIS need from me right now? What action should I take?"**

---

## 2. Core Screens

### Screen 1: Command Center (Home/Today)

**Purpose:** Everything you need to know and do today, in one glance.

**Layout:**
```
Header (Fixed)
├─ JARVIS logo + time
├─ Quick actions: [DONNA HELP] [SETTINGS]
└─ Status: "62% productivity today" 🟢

Critical Section (Urgent)
├─ Red alerts (action required immediately)
├─ Yellow warnings (needs attention soon)
└─ Each with 1-3 action buttons

Priorities Section (What to do next)
├─ AI-ranked tasks
├─ Time estimates (actual vs estimated from history)
├─ Recovery/energy context
├─ [START] [SCHEDULE] [SKIP] buttons

Quick Panels (Scrollable)
├─ 📅 Calendar (3-day snapshot)
├─ 💰 Finance (key alerts)
├─ 👥 People (contacts due)
├─ 💪 Health (recovery + recommendation)
├─ 📊 Productivity (today's metrics)
└─ 🧠 Donna's insight (what's important)

Footer (Fixed)
├─ [MENU] [NEW TASK] [DONNA HELP]
└─ Navigation tabs: Today | Tasks | Calendar | Health | Finance | People | Productivity | Critic
```

**Detailed View:**

```
╔════════════════════════════════════════════════════════╗
║  JARVIS - Command Center              Today 9:15 AM   ║
║                    [⚙️ HELP]                           ║
╚════════════════════════════════════════════════════════╝

⚡️  CRITICAL (Need action from you)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🔴 OVERDUE: Email VP about Q1 metrics (3 days late)
   ├─ Impact: High (boss waiting)
   ├─ Time: 15 min
   ├─ Recovery context: You're at 62/100 (focused work OK)
   └─ [SEND NOW] [DONNA DRAFT] [SNOOZE 2H]

🔴 BILL DUE: AWS subscription (2 days left) - $245
   ├─ Auto-pay setup? No
   └─ [PAY NOW] [SNOOZE] [DONNA PAY IT]

🟡 DECISION NEEDED: Accept Rahul's meeting invite?
   ├─ Time: Wed 3pm (1 hour)
   ├─ Conflict: None
   ├─ Donna analysis: "He's overdue contact, meeting is good"
   └─ [YES] [NO] [DONNA DECIDE]


✓ YOUR PRIORITIES TODAY (AI Ranked)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

1️⃣  Review metrics report
   ├─ Est: 45 min | Your avg: 42 min | Deadline: None
   ├─ Recovery: 62/100 (good for focused work) ✅
   ├─ Productivity: 78% this week (above avg) ✅
   ├─ Priority: High (blocks other work)
   └─ [START NOW] [SCHEDULE 11AM] [SKIP]

2️⃣  Close Q1 billing
   ├─ Est: 90 min | Your avg: 75 min | Deadline: 5pm TODAY ⏰
   ├─ Blocking: 2 other tasks
   ├─ Productivity: Usually 70% on finance tasks
   └─ [START NOW] [SCHEDULE 1PM] [DELEGATE]

3️⃣  Respond to 8 emails (CRITICAL: 2)
   ├─ Est: 20 min total | 3 min critical
   ├─ Donna: "Can respond to 5 non-critical, you do 3"
   ├─ Productivity: Email usually 95% efficient
   └─ [SHOW CRITICAL] [I'LL HANDLE ALL] [DONNA RESPOND]


📅 SCHEDULE (Next 3 days)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

TODAY (Sat) - 📊 Light day (4h meetings, 4h free)
├─ 10:00 AM - Sarah meeting (Prep: Ready ✓)
│           [OPEN PREP NOTES] [MEETING LINK]
├─ 2:00 PM  - FREE (Donna: Good time for deep work)
└─ 7:00 PM  - Dinner with friends

TOMORROW (Sun) - 📊 Heavy day (6h meetings)
├─ ⚠️  Donna: "High load, skip workout"
├─ 9:00 AM - Dentist [REMINDER SET]
├─ 3:00 PM - Investor call [PREP NEEDED]
└─ [CONFIRM] [RESCHEDULE] [DONNA HELP]

MONDAY (Apr 7)
├─ 6:00 AM - Workout (Recovery good, go hard!)
│           [SCHEDULE] [SKIP]
├─ 9:00 AM - FREE (Donna: Schedule CEO call here?)
│           [PROPOSE MEETING] [BLOCK TIME]
└─ 2:00 PM - Standup + follow-up [PREP: 10 min ready]

[VIEW FULL CALENDAR] [SUGGEST OPTIMAL SCHEDULE]


💰 FINANCES (Auto-synced, pending your review)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Portfolio: ₹5,012,500 ▲
├─ Today: +₹12,500 (+1.2%) ✅
├─ Month: +₹120,000 (+2.5%) ✅
└─ [VIEW HOLDINGS] [TRADING]

⚠️  ALERT: Spending +30% this week vs baseline
├─ Categories: Food ↑40% | Shopping ↑60% | Other ↓10%
├─ Analysis: Weekend overspending? Or actual change?
└─ [REVIEW DETAILS] [SET BUDGET] [DONNA ANALYZE]

💡 Action available: $5,000 ready to invest
├─ Suggestion: Move to index fund (historical avg return)
├─ Donna confidence: 85% (you usually agree)
└─ [APPROVE] [REVIEW OPTIONS] [DONNA DECIDE]

[FULL FINANCE TAB]


👥 RELATIONSHIPS (Smart reminders)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🔴 OVERDUE: Rahul (Friend) - 18 days since contact
├─ Target frequency: Every 14-21 days
├─ Last interaction: "Caught up on new project"
├─ Suggested message: "Hey! Been a while. How's X going?"
├─ Donna: "He usually responds within 2 hours"
└─ [SEND MESSAGE] [CALL] [SCHEDULE COFFEE] [SNOOZE]

🟡 BIRTHDAY ALERT: Mom (Family) - 5 days away
├─ Last gift: ₹5,000 (2 yrs ago)
├─ Donna: "Usually gift ₹5-10k, she loves books"
└─ [PLAN GIFT] [REMIND LATER] [DONNA HANDLE]

🟢 ON TRACK: Sarah (Work) - Contacted 3 days ago ✓

[VIEW ALL CONTACTS] [ADD PERSON]


💪 HEALTH & RECOVERY
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Recovery Score: 62/100 🟢 (Good)
├─ Sleep: 7.2 hours ✅ (target: 7-9h)
├─ HRV: 45 ms (normal for you)
├─ Mood: 7/10 🟢 (good)
└─ Resting HR: 58 bpm ✅

Workout streak: 8 days 🔥
├─ Last: Yesterday, 60 min strength
├─ Calorie burn: 400 cal
└─ Today's recommendation: Moderate (60 min strength)

⚠️  TOMORROW HEAVY: 6 meetings scheduled
├─ Donna: "High mental load coming"
├─ Recommendation: Light activity today (rest muscles)
├─ Target: Ready to go hard Monday
└─ [SKIP GYM TODAY] [LIGHT YOGA] [GO ANYWAY]

[FULL HEALTH] [LOG MOOD] [LOG WORKOUT]


📊 PRODUCTIVITY TODAY
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Today's Score: 62% 🟡 (Below your avg 75%)
├─ Tasks started: 3
├─ Tasks completed: 1 (Review metrics - on schedule)
├─ Time blocked: 4 hours
├─ Deep work sessions: 0 (40 min uninterrupted minimum)
├─ Distractions: Moderate (8 interruptions)
└─ Emails checked: 12 times (vs optimal 3-4)

This week:
├─ Average: 72% (down from 78% last week)
├─ Best day: Wed (84%)
├─ Worst day: Mon (58%)
├─ Pattern: Declining through week (rest needed?)

Donna's analysis: "You're losing focus. 
Emails are distracting. Try focus block mode tomorrow."
└─ [ENABLE FOCUS MODE] [IGNORE] [CONFIGURE]

[FULL PRODUCTIVITY INSIGHTS]


🧠 DONNA'S DAILY BRIEFING
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

"Good morning! You've got a moderate day ahead. 
Recovery is solid, so you can do focused work.

First: That VP email is 3 days overdue - 
knock that out now while you're fresh.

Then: Q1 billing needs 90 min (due 5pm) - 
block 1-3pm for it.

Health tip: Tomorrow is heavy (6 meetings). 
Light activity today to recover.

Financial note: Spending is up 30% - 
want to review where it's going?

One thing: Rahul is definitely due a contact. 
Let's reach out today?

You're at 62% productivity (below avg). 
Focus mode would help."

[ACKNOWLEDGE] [HELP ME PRIORITIZE] [DONNA TAKE OVER]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

[MENU] [NEW TASK] [⭐ DONNA HELP ME] [FOCUS MODE]
```

---

### Screen 2: Tasks Management

**Purpose:** Detailed view of all tasks, AI-ranked, with productivity insights.

```
╔════════════════════════════════════════════════════════╗
║  TASKS - Smart Work List           62% complete today ║
║  [↑ Priority] [↓ Time] [🔄 My pattern] [TODAY]         ║
╚════════════════════════════════════════════════════════╝

FILTER: [ALL] [TODAY] [OVERDUE] [BLOCKED]

🔴 IN PROGRESS (2)
───────────────────────────────────────────────────────

Review metrics report
├─ Started: 9:45 AM (2 min ago)
├─ Est: 45 min | Elapsed: 2 min | Remaining: ~43 min
├─ Recovery: 62/100 (good for this task) ✅
├─ Interruptions: 1 email (dismiss or handle?)
├─ [MARK DONE] [PAUSE] [EXTEND TIME] [NEED HELP]
└─ Time tracking: ON ⏱️

Q1 billing
├─ Status: Not started
├─ Est: 90 min | Deadline: 5:00 PM (8h left)
├─ Blocker: Need metrics report first (above)
├─ Productivity pattern: You're 75 min avg (est 90 min)
├─ [START NOW] [SCHEDULE 1PM] [DELEGATE TO DONNA]
└─ Time tracking: Ready ⏱️


⚪ TODAY (5 total, 3 critical)
───────────────────────────────────────────────────────

1. Email VP (CRITICAL) 
   ├─ Status: Not started (OVERDUE 3 days)
   ├─ Time: 15 min
   ├─ Productivity: Usually email is 95% efficient
   ├─ Donna: "Do this first, blocking other work"
   └─ [START NOW] [DONNA DRAFT] [SNOOZE]

2. Respond to 8 emails
   ├─ Critical: 2 | Non-critical: 6
   ├─ Donna: "I can handle 6, you do critical 2"
   └─ [I'LL HANDLE ALL] [LET DONNA RESPOND] [REVIEW]

3. Call investor (BLOCKED by metrics report)
   ├─ Est: 30 min
   ├─ Deadline: Before 5 PM
   ├─ Prep: 10 min (notes ready)
   └─ [SCHEDULE 2PM] [SKIP] [DELEGATE]

4. Update Notion database
   ├─ Est: 20 min
   ├─ Priority: Medium
   └─ [SCHEDULE] [SKIP] [DELEGATE]

5. Plan tomorrow's meetings
   ├─ Est: 15 min
   ├─ Priority: Medium (6 meetings to prepare for)
   └─ [SCHEDULE] [DONNA HELP] [SKIP]


🟢 UPCOMING (This week: 12 total)
───────────────────────────────────────────────────────

Tomorrow (6 tasks)
├─ Prepare for investor call
├─ Prepare for team meeting
├─ ... and 4 more

[VIEW WEEK] [AI SUGGEST SCHEDULE]


📊 YOUR TASK PATTERNS (Donna learns)
───────────────────────────────────────────────────────

Task accuracy: You estimate 90 min, average 75 min
└─ Donna adjustment: Multiply estimate by 0.83x

Best time for deep work: 10 AM - 12 PM
└─ Productivity: Usually 85%+ in this window

Email efficiency: 95% (very efficient)
└─ Donna: Batch emails when possible

Distractions: 8+ interruptions today (above avg 4)
└─ Recommendation: Enable focus mode (block all distractions)

[ENABLE FOCUS MODE] [VIEW FULL PATTERNS]


[BULK ACTIONS] [ARCHIVE COMPLETED] [ADD TASK]
```

---

### Screen 3: Calendar with AI Context

**Purpose:** Schedule with productivity, health, and relationship insights baked in.

```
╔════════════════════════════════════════════════════════╗
║  CALENDAR - Smart Schedule            April 2026       ║
║  [DAY] [WEEK] [MONTH] [AI SUGGESTIONS]                 ║
╚════════════════════════════════════════════════════════╝

SAT, APR 5 (TODAY) - Light day, 4h meetings
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

6:00 AM
├─ Workout ready (recovery 62/100 = moderate intensity)
├─ Donna: "Light activity today (tomorrow is heavy)"
└─ [SCHEDULE WORKOUT] [SKIP]

9:00 AM - FREE (3 hours) 🟢
├─ Donna: "Prime time for deep work (metrics report)"
├─ Productivity: Best focus hours 10 AM - 12 PM
├─ [BLOCK TIME] [SUGGEST TASK]

10:00 AM - Sarah meeting
├─ Duration: 1 hour
├─ Prep: Ready ✓
├─ Notes: [OPEN] [RESCHEDULE] [MARK DONE]
├─ Follow-up needed? (Donna checks)
└─ Post-meeting: Time to work on Q1 billing

2:00 PM - FREE (3 hours) 🟡
├─ Donna: "Good for admin tasks"
├─ Suggested: Q1 billing (90 min), emails (20 min)
├─ [SCHEDULE TASKS] [DONNA HELP]

5:00 PM - DEADLINE: Q1 billing DUE
├─ ⏰ 3 hours until deadline
├─ [MARK TIME] [EXTEND DEADLINE]

7:00 PM - Dinner with friends
├─ Travel time: 30 min (auto-added to calendar)
├─ Prep: None needed
├─ Recovery impact: Social (good for well-being)
└─ [CONFIRM] [RESCHEDULE] [SKIP]


SUN, APR 6 - ⚠️  HEAVY DAY (6h meetings)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

6:00 AM
├─ Donna: "Heavy meeting day coming - skip gym"
├─ Light yoga instead (20 min, low exertion)
└─ [SKIP GYM] [SCHEDULE YOGA] [GO HARD ANYWAY]

9:00 AM - Dentist appointment
├─ Duration: 1 hour
├─ Location: Downtown (30 min travel)
├─ Impact: Can't do focused work (grogginess after)
└─ [RESCHEDULE] [CONFIRM]

10:30 AM - FREE (30 min buffer)

3:00 PM - Investor call
├─ Duration: 30 min
├─ Importance: 🔴 CRITICAL
├─ Prep: [OPEN PREP NOTES] - 10 min needed
├─ Meeting link: Ready
├─ Recovery impact: Mental energy high at this hour
└─ [CONFIRM] [RESCHEDULE] [PREP]

Rest of day: Meetings back-to-back

Donna analysis: "Your energy will be low by 5 PM.
Schedule calls early (9 AM - 2 PM window).
Skip workouts, take it easy tonight."

[RESCHEDULE TO LIGHTER PATTERN]


MON, APR 7 - 📊 Balanced (mixed tasks + meetings)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

6:00 AM - Workout
├─ Recovery: Excellent (rested yesterday)
├─ Recommendation: HARD workout (strength)
├─ Duration: 90 min
└─ [SCHEDULE] [SKIP]

9:00 AM - FREE (2 hours)
├─ Donna: "CEO call could go here (vs afternoon)"
├─ Time: 1 hour meeting + 30 min prep
├─ [PROPOSE TO CEO] [SCHEDULE INTERNALLY]

11:30 AM - Standup meeting
├─ Duration: 30 min
├─ Prep: Minimal

[VIEW FULL WEEK] [AI SUGGEST OPTIMAL SCHEDULE]
```

---

### Screen 4: Productivity Analytics

**Purpose:** Detailed productivity tracking, patterns, and recommendations.

```
╔════════════════════════════════════════════════════════╗
║  PRODUCTIVITY - Your Work Analytics     This Week      ║
║  [DAY] [WEEK] [MONTH] [QUARTER]                        ║
╚════════════════════════════════════════════════════════╝

TODAY'S SNAPSHOT
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Overall Score: 62% 🟡 (Your avg: 75%)

Tasks:
├─ Started: 3
├─ Completed: 1 (metrics report - on time)
├─ In progress: 1 (Q1 billing - on track)
├─ Pending: 1 (emails)
├─ Overdue: 1 (VP email - being done now)
└─ Efficiency: 33% (1/3 complete, but it's early)

Time tracking:
├─ Deep work: 0 sessions (should be 2-3)
├─ Focus blocks: 0 (40+ min uninterrupted)
├─ Distraction interruptions: 8 (avg: 4, HIGH)
├─ Email checks: 12 (optimal: 3-4, TOO MANY)
├─ Context switches: 6 (each costs 15 min focus)
└─ Wasted time to context switching: ~90 min

Health context:
├─ Recovery: 62/100 (should help focus)
├─ Mood: 7/10 (good)
├─ Sleep: 7.2h (above target)
└─ Caffeine: Had coffee at 7 AM (should peak 10-12 PM)

Time-of-day analysis:
├─ 8-10 AM: 70% productive (setup, email checks)
├─ 10 AM-12 PM: 85% productive (peak focus window) ✅
├─ 12-2 PM: 65% productive (post-lunch dip)
├─ 2-5 PM: Pending (in progress)
└─ Donna: "You perform best 10 AM - 12 PM"


THIS WEEK'S TRENDS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

┌─────┬─────┬─────┬─────┬─────┐
│ Mon │ Tue │ Wed │ Thu │ Fri │ (Sat in progress)
├─────┼─────┼─────┼─────┼─────┤
│ 58% │ 65% │ 84% │ 72% │ 68% │
└─────┴─────┴─────┴─────┴─────┘

Weekly avg: 72% (above your 75% target by 3%)

Pattern analysis:
├─ Monday: Always low (weekend recovery? 58%)
├─ Wednesday: Your best day (84%)
│  └─ Donna: "Fewer meetings, more deep work"
├─ Thursday: Drop after Wed (72%)
│  └─ Donna: "Meeting load increases"
└─ Friday: Declining (68%)
   └─ Donna: "Fatigue kicks in"

Recommendation: "Take it easier Monday, 
block Wed for deep work, 
reduce Thu meetings if possible"

[IMPLEMENT SUGGESTION] [VIEW DETAILS]


TASK ACCURACY
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

You estimate: 90 min
You actually take: 75 min (avg, 83% of estimate)

By category:
├─ Deep work: 70% accurate (usually underestimate)
├─ Email: 105% accurate (actually longer)
├─ Meetings: 100% accurate (booked time matches)
├─ Admin: 120% accurate (always longer than expected)
└─ Creative: 60% accurate (highly variable)

Donna adjustment:
├─ Deep work: Multiply estimate by 1.43x
├─ Email: Estimate is accurate (no adjustment)
├─ Admin: Multiply by 1.2x (adds 20%)
└─ Creative: Expect 40% variance

[APPLY ADJUSTMENTS TO NEW TASKS]


FOCUS PATTERNS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Deep work (40+ min uninterrupted):
├─ Frequency: 2-3 times/week (could be 4-5)
├─ Duration: Avg 90 min when it happens
├─ Success rate: 92% (complete task in one session)
├─ Context: Usually morning (10 AM - 12 PM)

Interruption sources:
├─ Slack: 40% of interruptions
├─ Email: 35%
├─ Calendar alerts: 15%
├─ In-person: 10%

Donna recommendation: "Enable Do Not Disturb 
when you start deep work. 
Batch email checks to 3-4 times daily."

[ENABLE FOCUS MODE] [CONFIGURE DND] [BATCH EMAIL CHECKS]


BREAKS & RECOVERY
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Breaks taken today: 1 (too few)
├─ Duration: 5 min
├─ Type: Quick water break (not enough)

Research shows: 15 min break every 90 min = 10% more productivity

You're missing: 2 breaks (should be 3 total)
└─ Recommendation: Take 15 min break at 12 PM

[TAKE BREAK NOW] [SCHEDULE BREAKS] [IGNORE]


THIS MONTH'S PERFORMANCE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

April (partial): 74% avg ↑ (vs Mar 68%)
├─ Best day: Apr 3 (84%)
├─ Worst day: Apr 1 (52%)
├─ Trend: Improving ✅

Contributing factors:
├─ Health improved (was 58/100 recovery, now 62)
├─ Fewer meetings in early April
├─ Better sleep (avg 7.1h, up from 6.8h)
└─ One difficult project (Q1 billing) in progress

[VIEW FULL HISTORY]


DONNA'S PRODUCTIVITY COACHING
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

"You're doing well. 72% this week beats your target.

Three things would help even more:

1. REDUCE INTERRUPTIONS
   You're checking email 12x daily (I checked 3x for you).
   Each check = 15 min to regain focus.
   Loss: 2+ hours/day to context switching.
   Fix: Batch to 8 AM, 12 PM, 5 PM (3 checks/day)
   Impact: +15-20% productivity

2. PROTECT DEEP WORK TIME
   You do best 10 AM - 12 PM.
   But you don't block it (interruptions happen).
   Fix: [ENABLE FOCUS MODE] during this window
   Impact: +5-10% on deep work tasks

3. BREAK PATTERN (Mon/Fri SLUMP)
   You drop to 58% Monday, 68% Friday.
   This tells me: You're fatigued Friday, 
   and ramping back up Monday.
   Fix: Lighter meetings Mon/Fri, 
        rest Friday afternoon
   Impact: Consistent 75%+ all week

Try these 3 things. Check back Friday?"

[IMPLEMENT ALL] [PICK ONE] [IGNORE]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

[EXPORT REPORT] [SHARE WITH COACH] [SETTINGS]
```

---

### Screen 5: Critic Workspace

**Purpose:** Evaluate ideas, plans, decisions with AI guidance.

```
╔════════════════════════════════════════════════════════╗
║  CRITIC - Idea Evaluation            Workspace         ║
║  [NEW CRITIQUE] [HISTORY]                              ║
╚════════════════════════════════════════════════════════╝

INPUT AREA
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

"Describe your idea, plan, or decision..."

[TEXT INPUT (multi-line)]
  Should I launch an AI course for founders?
  
[VOICE INPUT] 🎤 (or paste text)

Options:
├─ [Include my life context?] ✅ 
│  (Recovery, finance, calendar, etc.)
├─ [Category] [Startup idea ▼]
└─ [SUBMIT FOR CRITIQUE]


CRITIQUE OUTPUT (Example)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🟡 VERDICT: YELLOW (Risky, but doable with conditions)
Session saved: Apr 5, 2026 at 9:45 AM

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

CORE CRITIQUE (3-5 specific problems)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

1. RECOVERY IS LOW (62/100)
   You're recovering moderately. 
   Starting an intense project (course creation) 
   while managing current work = burnout risk.
   Last time you were here? 3 months ago 
   (same recovery score, different project).
   
   Action: Wait 2 weeks, get recovery to 70+.

2. YOU UNDERESTIMATE TIMELINES BY 50%
   Course estimate: 3 months
   Your pattern: Add 50% = 4.5 months
   Income gap: You can last 8 months on savings.
   Safe? Yes. Comfortable? No.
   
   Action: Budget 5 months, not 3.

3. AI COURSE MARKET IS SATURATED
   Launched this month: 47 AI courses (ProductHunt)
   Your differentiation: ?
   You have 0 validation from founders yet.
   
   Action: Before building, validate with 5+ potential students.

4. EXECUTION RISK (YOUR PATTERN)
   2 years ago: Launched 3 projects
   You shipped 1 (50% completion rate)
   Time to ship: Always +50% of estimate
   
   Action: Assume this will take 5 months + overruns.


ASSUMPTIONS YOU'RE MAKING (What could be wrong)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✗ "Founders want a course on this topic"
  → You haven't surveyed even 5 potential buyers

✗ "I can build this in 3 months"
  → Your past projects: +50% time, always
  → You estimate 90 min tasks, usually take 75 min
  → But complex projects? Wildly underestimated

✗ "I can sustain 60 h/week for 3 months"
  → Your recovery: 62/100 (moderate)
  → Your calendar: Already 50% booked
  → Last high-intensity project: You burned out week 8

✗ "This is the right time"
  → Your productivity: 72% (below 75% target)
  → Your schedule: Heavy for next 6 weeks
  → Your focus: Distracted (8 email checks/day)


MISSING INFORMATION
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Before saying yes, you need:

1. Customer validation
   - Talk to 10 potential founders
   - What problem does this solve?
   - Would they pay $X?

2. Competitive landscape
   - What courses exist?
   - What's your unfair advantage?
   - Why you vs. existing?

3. Clear timeline
   - Not "3 months" but "Week 1: X, Week 2: Y..."
   - Buffer for life (meetings, other work)

4. Revenue model
   - Price: $49? $99? $499?
   - Volume: 10 students? 100? 1000?
   - Break-even: Month X?


IF YOU MUST PROCEED (3 specific changes)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

1. VALIDATE FIRST (2 weeks)
   - Talk to 10 founders
   - Gauge interest + willingness to pay
   - Define MVS (Minimum Viable Syllabus)
   
   Only proceed if: 5+ say "yes, I'd take it"

2. TIMELINE REALISM (5 months, not 3)
   - Week 1-2: Refine content, record 20% of videos
   - Week 3-6: Record 50% more (60% total)
   - Week 7-10: Refine, edit, test with beta (80%)
   - Week 11-16: Launch, 1-2 iterations based on feedback
   - Buffer: 2 weeks for overruns
   
   Then launch to paid audience.

3. COMMITMENT CLARITY
   - Max time: 40h/week (not 60h)
   - Non-negotiables: Keep day job, maintain health
   - Exit condition: If recovery drops below 50, pause


PERSONAL CONTEXT (How this fits your life)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Right now:
├─ Recovery: Moderate (62) ⚠️
├─ Productivity: 72% (below target 75%)
├─ Calendar: Heavy (6 meetings coming up)
├─ Finance: Good runway (8 months)
├─ Relationships: Some people overdue contact (Rahul)
└─ Fitness: On streak (8 days)

This course would:
├─ Reduce recovery further (stress)
├─ Impact productivity (major time commitment)
├─ Make relationships harder (no time)
├─ Keep fitness going (probably skip)
└─ Risk: Burnout by Month 3

Verdict: Not ideal timing.
Better time: 2-3 months from now, 
after current projects close + recovery improves.


SUMMARY
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🟡 YELLOW VERDICT: Risky, but possible

The idea has merit (AI for founders = real need).
You have runway to fail (8 months savings).
But the execution risk is high (timing, recovery, timeline).

Recommendation: 
Validate first (2 weeks). 
Then reassess in 2 months when recovery is better.

Questions for you:
- Why launch NOW vs. 3 months from now?
- Have you talked to any potential students?
- What happens if it takes 6 months, not 3?

[ACCEPT THIS VERDICT] [DISAGREE] [GET MORE DETAILS]

[SHARE WITH FRIEND] [SAVE] [PRINT]
```

---

## 3. Action Buttons & Data Capture

### Every button logs data:

```
User clicks [START] on a task
  ↓
System logs:
{
  action_type: "task_started",
  task_id: "...",
  task_name: "Review metrics",
  estimated_minutes: 45,
  recovery_score: 62,
  productivity_score: 62,
  time_of_day: "9:45 AM",
  day_of_week: "Saturday",
  calendar_load: "light",
  interruptions_count: 0,
  timestamp: "2026-04-05T09:45:00Z"
}
  ↓
Agents react:
- Productivity Agent: "User starting task, tracking..."
- LifeAdmin Agent: "Task in progress, remove from pending"
- Wellness Agent: "Recovery good for focus, monitor interruptions"

User completes task in 42 minutes
  ↓
System logs:
{
  action_type: "task_completed",
  task_id: "...",
  estimated_minutes: 45,
  actual_minutes: 42,
  efficiency: "93%",
  quality: "high",
  completed_at: "2026-04-05T10:27:00Z"
}
  ↓
Productivity Agent learns:
- This user estimates 45, usually takes 42 min (93% accurate)
- Adjustment: Don't change estimate, accurate pattern
- Time-of-day pattern: Morning tasks are accurate (90%+)
```

---

## 4. Navigation & Structure

### Bottom Tab Navigation (Fixed)

```
[📅 TODAY]  [✓ TASKS]  [📊 PRODUCTIVITY]  [🧠 CRITIC]  [⚙️ MORE]
```

### "MORE" Tab Menu

```
├─ 💰 Finance
├─ 📖 Content (News feed)
├─ 👥 People & Relationships
├─ 💪 Health & Recovery
├─ 📈 Analytics & Insights
├─ ⚙️ Settings
├─ 📞 Help & Donna
└─ 👤 Profile
```

---

## 5. Color & Visual Language

### Status Indicators

```
🔴 Red = Critical (action required NOW)
🟡 Yellow = Warning (needs attention soon)
🟢 Green = Good (on track)
⚪ Gray = Neutral (informational)

Recovery/Productivity/Health Rings:
├─ 0-25%: Red (take action)
├─ 25-50%: Orange (caution)
├─ 50-75%: Yellow (watch it)
├─ 75-100%: Green (good)
```

### Key Visual Elements

- **Rings**: Recovery score, productivity, task completion
- **Sparklines**: Trends (portfolio, productivity, recovery)
- **Progress bars**: Time until deadline, task progress
- **Heatmaps**: Productivity by day, mood by calendar
- **Charts**: Weekly trends, monthly patterns

---

## 6. Interaction Patterns

### Pattern 1: Action Button → Confirmation

```
User clicks [SEND EMAIL]
  ↓
System shows: "Ready to send (preview)"
[SEND] [EDIT] [DONNA WRITE FOR ME]
  ↓
User clicks [SEND]
  ↓
Email sent, logged
```

### Pattern 2: AI Suggestion → User Approval

```
Donna suggests: "Accept Rahul's meeting?"
[YES] [NO] [DONNA DECIDE]
  ↓
User clicks [YES]
  ↓
Meeting added, logged, Relationship agent updated
```

### Pattern 3: Alert → One-Click Action

```
"Bill due in 2 days"
[PAY NOW] [SNOOZE] [DONNA PAY IT]
  ↓
One click = action taken (data captured)
```

---

## 7. Mobile First Design

### Responsive Breakpoints

```
Mobile (< 600px):
├─ Full-screen cards
├─ Vertical stack
└─ Thumb-friendly buttons (56px min)

Tablet (600-1200px):
├─ 2-column layout
├─ Larger cards
└─ More white space

Desktop (1200px+):
├─ 3-column dashboard
├─ Side panels
└─ Rich charts
```

---

## 8. Dark Mode (Always On)

JARVIS is dark-first:
- 🌙 Easier on eyes (especially for tracking at night)
- 🎨 Premium feel
- ⚡ Battery efficient (especially AMOLED)

---

## 9. Empty States & Onboarding

### First Launch

```
╔════════════════════════════════════╗
║  Welcome to JARVIS                 ║
║  Your AI Operating System          ║
╚════════════════════════════════════╝

"I'm Donna, your AI assistant.
I'll manage your life, remember everything,
and help you make better decisions.

Let's get started..."

[CONNECT GOOGLE CALENDAR]
  ↓
[CONNECT NOTION]
  ↓
[CONNECT PLAID (Finance)]
  ↓
[ENABLE HEALTH TRACKING]
  ↓
[DONE - Let's go!]
```

### Onboarding Screens

- Set recovery targets
- Set productivity goals
- Add important people
- Configure notification timing
- Choose focus style (aggressive? relaxed?)

---

## 10. Keyboard Shortcuts & Power User Features

### Global Shortcuts

```
Cmd/Ctrl + K     = Quick action menu
Cmd/Ctrl + N     = New task
Cmd/Ctrl + S     = Search
Cmd/Ctrl + D     = Donna help
Cmd/Ctrl + F     = Focus mode
Cmd/Ctrl + T     = Time tracking
```

### Focus Mode

```
When enabled:
├─ Hide all notifications
├─ Close all chats/apps
├─ Show only current task
├─ Block: Slack, email, calendar alerts
├─ Allow: Timer, task notes, Donna emergency
└─ Duration: 90 min (customizable)

[PAUSE] [EXTEND 15 MIN] [END SESSION]
```

---

## 11. Real-Time Sync

### WebSocket Updates

Whenever something changes in real-time:
- Agent makes decision → UI updates immediately
- Task completed → productivity score updates
- Bill paid → Finance panel updates
- Contact reached → People urgency refreshes
- Recovery recalculated → Health ring refreshes

No refresh needed. Everything live.

---

## 12. Export & Integration

### User Can Export

- Daily/weekly/monthly reports
- All productivity data
- All decisions made
- All agent suggestions
- Full audit trail

### Integration Points

- Notion (save articles, tasks)
- Google Calendar (sync)
- Slack (send reminders)
- Email (forward articles)
- Apple Health (export data)

---

## 13. Voice Commands (iOS)

### Siri Integration

```
"Hey Siri, ask Jarvis: What should I do next?"
  ↓ Plays: "Based on your calendar and energy,
           start the metrics review. You're at
           peak focus time."

"Hey Siri, ask Jarvis: Log my workout"
  ↓ "Great workout! 60 min strength.
    Recovery will be 68 tomorrow."

"Hey Siri, ask Jarvis: Should I rest?"
  ↓ "Yes. You've been on 5 high-load days.
    Rest today, resume Monday."
```

---

## 14. Summary

This is not a dashboard. This is a **real operating system for your life**.

Key principles:
- ✅ Action-driven (buttons everywhere)
- ✅ Data-logged (every action = data for learning)
- ✅ AI-assisted (Donna suggests, you confirm)
- ✅ Beautiful (dark, clean, premium)
- ✅ Proactive (tells you what to do)
- ✅ Personal (learns from your patterns)
- ✅ Integrated (syncs with all your tools)
- ✅ Fast (no friction, one-click actions)

This is something people will **actually use every day** because it manages their life better than they do.

