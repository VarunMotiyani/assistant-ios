# JARVIS — Database Schema

## PostgreSQL Tables

### users
```sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email TEXT UNIQUE NOT NULL,
  name TEXT,
  apns_device_token TEXT,
  timezone TEXT DEFAULT 'Asia/Kolkata',
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now()
);
```

### user_preferences
```sql
CREATE TABLE user_preferences (
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  key TEXT NOT NULL,
  value JSONB,
  PRIMARY KEY (user_id, key)
);
-- Keys: notification_times, supplement_targets, savings_rate_target,
--       emergency_fund_target, monthly_income, content_topic_weights
```

### user_goals
```sql
CREATE TABLE user_goals (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  category TEXT, -- startup, fitness, finance, learning, personal
  title TEXT,
  description TEXT,
  target_date DATE,
  status TEXT DEFAULT 'active', -- active, achieved, abandoned
  created_at TIMESTAMPTZ DEFAULT now()
);
```

***

## Life Admin

### tasks
```sql
CREATE TABLE tasks (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  notion_page_id TEXT,
  title TEXT NOT NULL,
  category TEXT, -- work, personal, finance, health, learning, startup
  priority TEXT, -- critical, high, medium, low
  status TEXT DEFAULT 'not_started', -- not_started, in_progress, done
  due_date DATE,
  notes TEXT,
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now()
);
```

### bills
```sql
CREATE TABLE bills (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  name TEXT NOT NULL,
  amount_inr NUMERIC,
  due_date DATE,
  recurrence TEXT, -- monthly, quarterly, annual, one_time
  status TEXT DEFAULT 'unpaid', -- unpaid, paid, auto_debit
  category TEXT, -- subscription, utility, insurance, emi, other
  created_at TIMESTAMPTZ DEFAULT now()
);
```

### briefings
```sql
CREATE TABLE briefings (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  date DATE NOT NULL,
  top_tasks JSONB,
  critical_alerts JSONB,
  wellness_summary JSONB,
  finance_summary JSONB,
  top_stories JSONB,
  reach_outs JSONB,
  critic_insight TEXT,
  generated_at TIMESTAMPTZ DEFAULT now()
);
```

***

## Wellness

### health_snapshots
```sql
CREATE TABLE health_snapshots (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  date DATE NOT NULL,
  sleep_duration_minutes INT,
  sleep_efficiency_pct NUMERIC,
  sleep_deep_minutes INT,
  sleep_rem_minutes INT,
  hrv_ms NUMERIC,
  resting_hr_bpm INT,
  steps INT,
  active_calories INT,
  stand_hours INT,
  body_weight_kg NUMERIC,
  recovery_score INT, -- 0-100 computed
  workload_score INT, -- 0-100 computed
  synced_at TIMESTAMPTZ DEFAULT now(),
  UNIQUE(user_id, date)
);
```

### mood_logs
```sql
CREATE TABLE mood_logs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  logged_at TIMESTAMPTZ DEFAULT now(),
  score INT CHECK (score BETWEEN 1 AND 10),
  note TEXT,
  energy_level INT CHECK (energy_level BETWEEN 1 AND 10)
);
```

### workouts
```sql
CREATE TABLE workouts (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  healthkit_uuid TEXT UNIQUE,
  workout_type TEXT,
  start_time TIMESTAMPTZ,
  end_time TIMESTAMPTZ,
  duration_minutes INT,
  calories_burned INT,
  avg_heart_rate_bpm INT,
  notes TEXT
);
```

### supplement_logs
```sql
CREATE TABLE supplement_logs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  date DATE NOT NULL,
  supplement TEXT, -- creatine, whey_post, whey_second
  logged_at TIMESTAMPTZ DEFAULT now(),
  UNIQUE(user_id, date, supplement)
);
```

### hydration_logs
```sql
CREATE TABLE hydration_logs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  date DATE NOT NULL,
  amount_ml INT,
  logged_at TIMESTAMPTZ DEFAULT now()
);
```

***

## Finance

### holdings
```sql
CREATE TABLE holdings (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  symbol TEXT NOT NULL,
  name TEXT,
  asset_type TEXT, -- equity, mutual_fund, etf, gold, fd, crypto, cash
  quantity NUMERIC,
  avg_cost_price NUMERIC,
  current_price NUMERIC,
  current_value NUMERIC,
  unrealized_pnl NUMERIC,
  currency TEXT DEFAULT 'INR',
  last_updated TIMESTAMPTZ DEFAULT now(),
  UNIQUE(user_id, symbol)
);
```

### transactions
```sql
CREATE TABLE transactions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  date DATE NOT NULL,
  description TEXT,
  amount_inr NUMERIC,
  transaction_type TEXT, -- income, expense, investment, transfer
  category TEXT,
  account TEXT,
  created_at TIMESTAMPTZ DEFAULT now()
);
```

### net_worth_snapshots
```sql
CREATE TABLE net_worth_snapshots (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  date DATE NOT NULL,
  total_portfolio_value NUMERIC,
  total_cash NUMERIC,
  total_net_worth NUMERIC,
  monthly_savings_rate NUMERIC,
  UNIQUE(user_id, date)
);
```

***

## Content

### feed_items
```sql
CREATE TABLE feed_items (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  source TEXT, -- techcrunch, hackernews, arxiv, producthunt, etc.
  url TEXT UNIQUE NOT NULL,
  title TEXT,
  summary TEXT,
  idea_card JSONB, -- full IdeaCard object if generated
  relevance_score NUMERIC,
  published_at TIMESTAMPTZ,
  fetched_at TIMESTAMPTZ DEFAULT now(),
  tags TEXT[]
);
```

### user_feed_interactions
```sql
CREATE TABLE user_feed_interactions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  feed_item_id UUID REFERENCES feed_items(id),
  action TEXT, -- viewed, saved, skipped, liked
  interacted_at TIMESTAMPTZ DEFAULT now(),
  notion_page_id TEXT
);
```

***

## Relationships

### contacts
```sql
CREATE TABLE contacts (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  notion_page_id TEXT,
  name TEXT NOT NULL,
  relationship_type TEXT, -- family, close_friend, friend, professional, acquaintance
  birthday DATE,
  important_date DATE,
  important_date_label TEXT,
  last_contacted DATE,
  contact_frequency TEXT, -- weekly, biweekly, monthly, quarterly
  importance TEXT DEFAULT 'medium', -- high, medium, low
  notes TEXT,
  tags TEXT[],
  urgency_score NUMERIC, -- computed by agent
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now()
);
```

### contact_interactions
```sql
CREATE TABLE contact_interactions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  contact_id UUID REFERENCES contacts(id),
  interacted_at TIMESTAMPTZ DEFAULT now(),
  channel TEXT, -- call, message, email, in_person
  notes TEXT
);
```

***

## Critic

### critic_sessions
```sql
CREATE TABLE critic_sessions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  prompt TEXT NOT NULL,
  context_snapshot JSONB, -- recovery, finance, calendar load at time of session
  verdict TEXT, -- red, yellow, green
  core_critique TEXT,
  assumptions TEXT,
  missing_info TEXT,
  if_you_must TEXT,
  personal_context TEXT,
  full_response TEXT,
  embedding vector(1536), -- pgvector for semantic search
  created_at TIMESTAMPTZ DEFAULT now()
);
```

***

## pgvector Index

```sql
CREATE INDEX ON critic_sessions USING ivfflat (embedding vector_cosine_ops) WITH (lists = 100);
```

***

## Redis Key Patterns

| Key | Type | TTL | Purpose |
|---|---|---|---|
| `briefing:{user_id}:{date}` | JSON string | 24h | Cached daily briefing |
| `feed:{user_id}:latest` | JSON string | 6h | Latest curated feed |
| `healthsync:{user_id}` | JSON string | 1h | Last HealthKit payload |
| `ws:nudge:{user_id}` | PubSub channel | — | Real-time nudge push |
| `ratelimit:{user_id}:{endpoint}` | Counter | 60s | Rate limiting |