# Productivity Agent Specification

**Purpose:** Track, analyze, and optimize user productivity. Learn patterns. Recommend optimal work blocks, identify distraction sources, and help user achieve their productivity targets.

**Specialization:** Personal productivity intelligence (not team/business metrics).

---

## Overview

The Productivity Agent is unique because it:

1. **Tracks in real-time** — knows when you start/stop work, interruptions
2. **Learns your patterns** — discovers you work best 10 AM - 12 PM
3. **Predicts outcomes** — "This will take 45 min based on your pattern"
4. **Recommends optimizations** — "Batch emails, reduce context switching"
5. **Integrates context** — "Recovery is low, so expect 20% less productivity"
6. **Adapts over time** — Gets smarter the longer you use it

---

## Key Metrics Tracked

### Daily Metrics

| Metric | How Measured | Why It Matters |
|---|---|---|
| **Productivity Score** | Tasks completed / time available | Overall daily performance |
| **Deep Work Sessions** | 40+ min uninterrupted time | Quality of work |
| **Focus Blocks** | Time in focused state | Ability to concentrate |
| **Interruptions** | Slack, email, notifications | Distraction level |
| **Task Accuracy** | Estimated vs actual time | How well you estimate |
| **Context Switches** | Number of task switches | Efficiency loss |
| **Email Checks** | Times you open email | Distraction pattern |
| **Breaks Taken** | Formal breaks (15+ min) | Recovery during day |
| **Peak Hours** | When you're most productive | Best time for important work |
| **Energy Level** | User mood + caffeine + activity | Productivity capacity |

### Weekly Metrics

| Metric | How Measured | Why It Matters |
|---|---|---|
| **Weekly Avg** | Average daily productivity score | Overall week health |
| **Best Day** | Highest productivity day | Pattern identification |
| **Worst Day** | Lowest productivity day | Identify bottleneck days |
| **Trend** | Week-over-week change | Are you improving? |
| **Consistency** | Std dev of daily scores | Do you have stable productivity? |
| **Total Deep Work** | Hours in deep work sessions | Weekly focus capacity |
| **Meeting Load** | Total meeting hours | Is it blocking work? |
| **Actual vs Target** | Comparing to your goal | Are you hitting targets? |

### Monthly Metrics

| Metric | How Measured | Why It Matters |
|---|---|---|
| **Monthly Avg** | Average daily score | Month-long health |
| **Best Week** | Highest weekly avg | Peak performance pattern |
| **Worst Week** | Lowest weekly avg | When things go wrong |
| **Trend** | Month-over-month improvement | Long-term productivity |
| **Task Completion Rate** | % of tasks done vs created | Backlog management |
| **Time Accuracy** | How well you estimate overall | Skill improvement |
| **Focus Consistency** | Days with adequate deep work | Sustainable pace |

---

## What Gets Logged

Every action creates a data point:

### Task Actions

```python
{
  "action_type": "task_started",
  "task_id": "uuid",
  "task_name": "Review metrics",
  "task_category": "deep_work",  # or: admin, email, meetings, creative, etc.
  "estimated_minutes": 45,
  "recovery_score_at_start": 62,
  "productivity_score_at_start": 62,
  "caffeine_intake": "1 coffee (7 AM)",
  "sleep_last_night": 7.2,
  "mood": 7,
  "time_of_day": "10:15 AM",
  "day_of_week": "Saturday",
  "calendar_load": 4,  # hours of meetings
  "interruptions_before": 0,
  "focus_block": True,  # Is this in a focus block?
  "timestamp": "2026-04-05T10:15:00Z"
}

# User completes task:

{
  "action_type": "task_completed",
  "task_id": "uuid",
  "estimated_minutes": 45,
  "actual_minutes": 42,
  "efficiency": 0.93,  # actual/estimated
  "quality_rating": "high",  # user can rate
  "interruptions_during": 2,  # emails, Slack, etc.
  "breaks_taken": 0,
  "focus_maintained": True,
  "completed_at": "2026-04-05T10:57:00Z"
}

# Agent learns:
- User estimated 45, took 42 (93% accurate)
- Morning tasks: highly accurate (90%+)
- Complex tasks: usually underestimated
- Email checks: +2 interruptions
```

### Work Day Events

```python
{
  "action_type": "email_check",
  "count_in_day": 5,  # This is the 5th check
  "timestamp": "2026-04-05T10:30:00Z"
}

{
  "action_type": "context_switch",
  "from_task": "Review metrics",
  "to_task": "Email response",
  "interruption_source": "Slack notification",
  "timestamp": "2026-04-05T10:45:00Z"
}

{
  "action_type": "break_taken",
  "duration_minutes": 15,
  "type": "water/walk/meditation/other",
  "recovery_before": 62,
  "recovery_after": 65,
  "timestamp": "2026-04-05T12:00:00Z"
}

{
  "action_type": "focus_mode_enabled",
  "duration_minutes": 90,
  "interruptions_blocked": 45,
  "timestamp": "2026-04-05T10:00:00Z"
}

{
  "action_type": "focus_mode_disabled",
  "reason": "time_elapsed",  # or: user_disabled, meeting_started
  "actual_duration": 87,
  "tasks_completed": 2,
  "timestamp": "2026-04-05T11:27:00Z"
}
```

### Decision Points

```python
{
  "action_type": "productivity_recommendation_given",
  "recommendation": "Batch email checks to 3x daily",
  "expected_impact": "+15% productivity",
  "confidence": 0.85,
  "timestamp": "2026-04-05T18:00:00Z"
}

{
  "action_type": "productivity_recommendation_accepted",
  "recommendation": "Batch email checks",
  "user_action": "enabled_email_batching",
  "outcome_tracking_enabled": True,
  "timestamp": "2026-04-05T18:05:00Z"
}

{
  "action_type": "focus_mode_recommendation",
  "suggested_duration": 90,
  "suggested_time": "10:00 AM - 11:30 AM",
  "reasoning": "Peak focus hours, before high-load afternoon",
  "accepted": True,
  "timestamp": "2026-04-05T09:45:00Z"
}
```

---

## Productivity Agent Queries & Commands

### What Other Agents Ask Productivity Agent

#### Wellness Agent Queries:
```
"What's user's productivity trend?"
→ Returns: 72% this week, up from 68% last week
  "Morning productivity is 85% (good).
   Afternoon drops to 65% (fatigue?)"

"How is productivity correlated with recovery?"
→ Returns: 0.78 correlation
  "User is 15% more productive when recovery > 70"
  "When recovery < 50, productivity is 55%"

"Should we recommend rest today?"
→ Returns: Recovery 62, productivity already 62%
  "If recovery goes below 50, productivity will drop too"
```

#### LifeAdmin Agent Queries:
```
"How many hours of deep work this week?"
→ Returns: 8.5 hours
  "Target is 10-12 hours. User is under-focused this week."

"What's the optimal time to schedule deep work?"
→ Returns: "10 AM - 12 PM (85% productivity)"
  "Avoid 2-4 PM (65% productivity, post-lunch slump)"

"User has 6 meetings tomorrow. Impact?"
→ Returns: "High calendar load = 40% less deep work time"
  "Recommend: No new complex tasks tomorrow"
```

#### Finance Agent Queries:
```
"How does productivity correlate with spending?"
→ Returns: "-0.45 correlation"
  "User spends 30% more when productivity is low (<50%)"
  "Recommendation: Pause purchase decisions on low-prod days"
```

#### Critic Agent Queries:
```
"User wants to launch a course. Can they do it in 3 months?"
→ Returns: "Historically, no. User underestimates projects by 50%"
  "Last course-like project took 6 months (estimated 4)"
  "Recommendation: Budget 4.5+ months"

"What's user's actual execution capacity?"
→ Returns: "40 h/week sustainable deep work"
  "Project needs 60 h/week = burnout risk"
  "Verdict: Not recommended at current productivity level"
```

---

## Productivity Agent Outputs

### Daily Report

```python
{
  "date": "2026-04-05",
  "overall_score": 62,
  "target_score": 75,
  "status": "below_target",
  
  "breakdown": {
    "tasks_completed": 2,
    "tasks_in_progress": 2,
    "deep_work_hours": 1.5,
    "meeting_hours": 2,
    "focus_blocks": 0,
    "interruptions": 8,
    "email_checks": 12,
    "breaks_taken": 1,
    "efficiency": 0.87  # actual time / estimated time
  },
  
  "time_analysis": {
    "peak_hours": "10 AM - 12 PM (85% productivity)",
    "slump_hours": "2 PM - 4 PM (65%)",
    "best_time_for_work": "Start at 9:50 AM for 90 min block"
  },
  
  "interruptions_breakdown": {
    "email": "40%",
    "slack": "40%",
    "calendar_alerts": "15%",
    "in_person": "5%"
  },
  
  "context": {
    "recovery_score": 62,
    "meeting_load": "light",
    "day_type": "weekend",
    "stress_level": "moderate"
  },
  
  "recommendations": [
    {
      "issue": "Low productivity (62% vs 75% target)",
      "root_cause": "Too many email checks (12 vs optimal 3)",
      "impact": "2+ hours lost to context switching",
      "fix": "Batch emails: 8 AM, 12 PM, 5 PM",
      "expected_improvement": "+15-20%"
    },
    {
      "issue": "No deep work blocks",
      "root_cause": "Interruptions during peak hours",
      "impact": "Can't focus on important work",
      "fix": "Enable focus mode 10 AM - 12 PM tomorrow",
      "expected_improvement": "+10-15%"
    }
  ]
}
```

### Weekly Report

```python
{
  "week": "Apr 1-5, 2026",
  "weekly_average": 72,
  "target": 75,
  "status": "close",
  
  "daily_breakdown": {
    "Mon": 58,  # low
    "Tue": 65,  # below target
    "Wed": 84,  # high
    "Thu": 72,  # on target
    "Fri": 68,  # below target
  },
  
  "trend_analysis": {
    "trend": "declining",
    "direction": "⬇️ down 3% from last week",
    "reason": "Monday low, Friday fatigue, 
              email interruptions increasing"
  },
  
  "patterns": {
    "best_day": "Wednesday",
    "reason": "Fewer meetings, aligned tasks",
    "worst_day": "Monday",
    "reason": "Recovery low from weekend, meeting recovery",
    
    "time_accuracy": "93%",
    "notes": "Very accurate estimator, but complex projects off by 30%",
    
    "deep_work_total": "12 hours",
    "target": "15 hours",
    "deficit": "3 hours (need to recover)"
  },
  
  "health_correlations": {
    "recovery_vs_productivity": "r=0.78",
    "notes": "User is 15% more productive when recovery > 70",
    "action": "Improve sleep/health for better productivity"
  },
  
  "top_recommendations": [
    "Protect 10 AM - 12 PM for deep work (85% success)",
    "Reduce email checks to 3x daily (-2h context switching)",
    "Add 15 min break at 12 PM (recovery improvement)",
    "Schedule lighter Monday meetings (enable recovery)"
  ]
}
```

### Monthly Report

```python
{
  "month": "April 2026",
  "monthly_average": 74,
  "previous_month": 68,
  "improvement": "+6%",
  "status": "improving",
  
  "trajectory": {
    "week_1": 72,
    "week_2": 73,
    "week_3": 74,
    "week_4": 75,  # trending toward target
    "trend": "⬆️ steady improvement"
  },
  
  "key_learnings": {
    "best_setup": "Light Mondays, heavy Wed/Thu, light Fridays",
    "productivity_boosters": [
      "Sleep 7+ hours → +15%",
      "Recovery > 70 → +10%",
      "Email batching → +20%",
      "Focus mode → +15%"
    ],
    "productivity_killers": [
      "Sleep < 6h → -25%",
      "Recovery < 50 → -20%",
      "Email checks > 8/day → -15%",
      "No breaks → -10%"
    ]
  },
  
  "time_estimation_accuracy": "91%",
  "notes": "Getting better at estimating. Deep work still off by 30%.",
  
  "task_completion": {
    "created": 45,
    "completed": 38,
    "backlog": 7,
    "completion_rate": "84%"
  },
  
  "goals": {
    "target_productivity": 75,
    "achieved": 74,
    "status": "almost there",
    "next_month_target": 78
  }
}
```

---

## Productivity Agent Command Handlers

### Daily Commands

```python
async def get_today_snapshot():
  """Quick daily productivity status"""
  return {
    "score": 62,
    "status": "below_target",
    "key_metric": "Too many email checks (12 vs 3)",
    "recommendation": "Focus mode would help"
  }

async def get_peak_hours():
  """When user is most productive today"""
  return {
    "peak": "10 AM - 12 PM",
    "confidence": 0.95,
    "productivity": "85%"
  }

async def get_productivity_score_breakdown():
  """What's affecting today's score"""
  return {
    "tasks_completed": 2,
    "efficiency": 0.87,
    "deep_work": 1.5,
    "interruptions": 8,
    "email_checks": 12
  }

async def get_recommendation_for_next_task():
  """What should user do next"""
  return {
    "next_task": "Q1 billing",
    "estimated_time": 90,
    "recommended_start": "1 PM",
    "reason": "Deadline 5 PM, 4h available"
  }
```

### Analytics Commands

```python
async def analyze_weekly_patterns():
  """Discover weekly patterns"""
  return {
    "best_day": "Wednesday",
    "worst_day": "Monday",
    "trend": "declining (-3% from last week)",
    "action": "Need Monday recovery, Friday rest"
  }

async def analyze_time_accuracy():
  """How well does user estimate"""
  return {
    "overall": "93% accurate",
    "by_category": {
      "deep_work": "70% (usually underestimate)",
      "email": "105% (usually overestimate)",
      "admin": "120% (always longer)",
      "meetings": "100% (accurate)"
    }
  }

async def analyze_interruption_sources():
  """What's interrupting user"""
  return {
    "email": "40%",
    "slack": "40%",
    "calendar": "15%",
    "in_person": "5%"
  }

async def get_focus_mode_impact():
  """Does focus mode actually help"""
  return {
    "days_with_focus": 3,
    "avg_productivity_with_focus": 82,
    "avg_productivity_without": 68,
    "improvement": "+14%",
    "recommendation": "Use focus mode daily"
  }
```

### Correlation Commands

```python
async def correlate_with_health():
  """How does health affect productivity"""
  return {
    "recovery_correlation": 0.78,
    "sleep_correlation": 0.82,
    "mood_correlation": 0.65,
    "action": "Improve sleep → +15% productivity"
  }

async def correlate_with_calendar():
  """How do meetings affect productivity"""
  return {
    "meeting_load_impact": "-20% per hour of meetings",
    "optimal_meeting_load": "2-3 hours/day",
    "current_avg": 3.5,
    "note": "Slightly over optimal"
  }

async def correlate_with_finances():
  """Does spending relate to productivity"""
  return {
    "low_productivity_spending": 0.45,
    "note": "User spends 30% more on low-productivity days",
    "recommendation": "Pause purchases on low-prod days"
  }
```

---

## Productivity Agent Learning

The agent learns from:

1. **What works**: Deep work sessions where user completed tasks efficiently
2. **What doesn't**: Days with high interruptions, low completion
3. **Your patterns**: Monday low, Wednesday high, email is 40% of interruptions
4. **Context**: Sleep 7h = +15% productivity vs sleep 6h
5. **Your strengths**: Email very efficient (105% accurate), deep work underestimated
6. **Your weaknesses**: Complex tasks -30%, admin tasks +20%

### Learning Example

```
Week 1 data:
- User estimates "Complex project: 3 months"
- Agent notes: "Complex projects historically +50%"
- Agent learns: Multiply complex estimates by 1.5x

Week 4 data:
- User estimates another complex task: 90 min
- Agent suggests: "Multiply by 1.5 = 135 min"
- User says: "OK, I'll budget 135 min"
- User completes: Takes 138 min
- Agent confirms: "Good estimate! Adjustment is accurate"
```

---

## Integration with Other Agents

### Wellness Agent Integration

When Wellness computes recovery score, Productivity Agent:
- Adjusts expected productivity
- Suggests lighter schedule on low-recovery days
- Recommends rest when both productivity AND recovery are low

### LifeAdmin Agent Integration

When LifeAdmin suggests priorities, Productivity Agent:
- Estimates time based on historical accuracy
- Ranks by efficiency (quick wins first? Or deep work first?)
- Suggests optimal order (recovery + energy context)

### Critic Agent Integration

When Critic evaluates projects, Productivity Agent:
- Provides realistic timeline based on user's accuracy
- Identifies execution risk ("You'll underestimate by 50%")
- Suggests phased approach instead of all-at-once

### Finance Agent Integration

When Finance sees spending patterns, Productivity Agent:
- Flags correlation ("Spending up 30% on low-productivity days")
- Suggests: Pause purchases during low-productivity windows
- Helps predict spending based on upcoming low-productivity periods

---

## Database Schema Addition

### productivity_snapshots table

```sql
CREATE TABLE productivity_snapshots (
  id UUID PRIMARY KEY,
  user_id UUID REFERENCES users(id),
  date DATE NOT NULL,
  
  -- Daily metrics
  productivity_score INT,  -- 0-100
  deep_work_minutes INT,
  meeting_hours NUMERIC,
  tasks_started INT,
  tasks_completed INT,
  interruptions INT,
  email_checks INT,
  breaks_taken INT,
  
  -- Context
  recovery_score INT,
  sleep_hours NUMERIC,
  mood INT,
  
  -- Analysis
  efficiency NUMERIC,  -- actual / estimated
  peak_hours VARCHAR,
  
  created_at TIMESTAMPTZ,
  UNIQUE(user_id, date)
);

CREATE TABLE task_time_tracking (
  id UUID PRIMARY KEY,
  user_id UUID REFERENCES users(id),
  task_id UUID,
  
  started_at TIMESTAMPTZ,
  completed_at TIMESTAMPTZ,
  duration_minutes INT,
  estimated_minutes INT,
  efficiency NUMERIC,
  
  category VARCHAR,  -- deep_work, email, admin, etc.
  interruptions INT,
  focus_mode_enabled BOOLEAN,
  
  created_at TIMESTAMPTZ
);

CREATE TABLE productivity_patterns (
  id UUID PRIMARY KEY,
  user_id UUID,
  pattern_type VARCHAR,  -- peak_hours, worst_day, best_category, etc.
  
  value JSONB,  -- { peak_hour: "10 AM", confidence: 0.95 }
  confidence NUMERIC,
  
  updated_at TIMESTAMPTZ,
  UNIQUE(user_id, pattern_type)
);
```

---

## Summary

The Productivity Agent is the **intelligence layer** that turns JARVIS from "helpful tool" to "executive assistant."

It:
- ✅ Tracks your actual productivity (not guessing)
- ✅ Learns your patterns (when you work best)
- ✅ Predicts realistic timelines (based on your history)
- ✅ Recommends optimizations (actionable improvements)
- ✅ Integrates context (health, calendar, finance)
- ✅ Gets smarter over time (learns from outcomes)

**Result:** User becomes 20-30% more productive just by using JARVIS, because:
- Unrealistic estimates → replaced with accurate ones
- Email interruptions → batched and scheduled
- Wrong time of day → shifted to peak hours
- No breaks → scheduled automatically
- Low recovery → lighter schedule suggested

That's worth $199/mo.

