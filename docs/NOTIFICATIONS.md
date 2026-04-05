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
    actions: [1,2,3,4,5,6,7,8,9,10].map { score in
        UNNotificationAction(identifier: "MOOD_\(score)", title: "\(score)")
    },
    intentIdentifiers: []
)
```

**Post-Workout Whey:**
```swift
UNNotificationCategory(
    identifier: "WHEY_LOG",
    actions: [
        UNNotificationAction(identifier: "WHEY_YES", title: "✅ Logged"),
        UNNotificationAction(identifier: "WHEY_SKIP", title: "Skip")
    ],
    intentIdentifiers: []
)
```

**Reach Out Confirmation:**
```swift
UNNotificationCategory(
    identifier: "REACH_OUT",
    actions: [
        UNNotificationAction(identifier: "CONTACTED", title: "✅ Done"),
        UNNotificationAction(identifier: "LATER", title: "Remind later"),
        UNNotificationAction(identifier: "VIEW", title: "See message")
    ],
    intentIdentifiers: []
)
```

### Local Notification Scheduling
All recurring daily notifications are scheduled as local notifications by the iOS app (no server dependency for basic nudges). Backend push (APNs) is used only for:
- Dynamic content (market alerts, breaking news, personalized briefings)
- Ad-hoc triggered events

Local notifications are rescheduled every night during the BGProcessingTask nightly sync.