# Ahimo Features

A complete list of what Ahimo does on iPhone, Apple Watch, and your home screen.

## Logging — fast context capture

- **Mood** logging with quick-tap scale (watch + phone)
- **Energy** level
- **Stress** level
- **Drinks** — alcoholic and caffeinated, with type, count, and time
- **Symptoms** — headache, nausea, soreness, custom symptoms
- **Intimacy**
- **Menstrual events** — period, flow, cycle phase
- **Tags** — free-text labels for anything else
- **Three-date model** — every entry stores when you logged it, when it happened, and which calendar day it belongs to (so a late-night drink correctly counts toward yesterday)

## HealthKit integration

- **Sleep** — duration, stages, schedule consistency
- **Heart rate** and **resting heart rate**
- **HRV** (heart rate variability)
- **Steps**, walking distance, active energy
- **Workouts** — read structured workouts from any source
- **Menstrual data** — bidirectional sync
- **Blood pressure**
- Single-file HealthKit boundary — only one file in the app touches HealthKit, by architectural rule, so the data path is auditable

## CSQI — Composite Sleep Quality Index

- A single 0–100 score that weighs **duration**, **schedule consistency**, and **stage composition**
- Per-night and rolling averages
- Drives the sleep insights and the Well-Being widget

## The Insights pipeline

### Layer 1 — Statistical baselines

- 7-day, 30-day, 90-day rolling averages
- Week-over-week deltas
- Percentile bands
- Computed on-demand when you open a detail view; never persisted

### Layer 2 — Correlation analysis (30 analyzers)

Background-generated, refreshed every 24 hours and on demand:

- **Sleep impact** — sleep consequences, symptom correlations, mood/energy effects, schedule consistency
- **Heart rate** — HRV ↔ sleep, HRV ↔ mood, HRV ↔ stress
- **Activity** — activity ↔ sleep, mood, stress, energy
- **Menstrual** (7 analyzers) — cycle phase ↔ sleep / symptoms / HRV / mood / stress / energy / intimacy
- **Behavioral** — weekday/weekend patterns, time-of-day patterns, mood-energy bidirectional, tag analysis
- **Lifestyle** — drinks ↔ sleep
- **Recovery** — composite recovery impact engine
- **Circadian** — circadian energy alignment
- **Absence** — detects when *not* doing something has an impact

Each insight is a plain-language sentence with a magnitude and direction. Severity-coded: green (positive), gray (neutral), yellow (attention), red (warning).

### Layer 3 — On-device LLM insights

- Weekly narrative insights generated from Layer 2 findings + your profile
- Two interchangeable backends:
  - **Apple Intelligence** (Foundation Models framework) on supported devices
  - **MLX** on any modern iPhone — defaults to `SmolLM3-3B-4bit`, with `Qwen3.5-4B` and `Llama-3.2-1B` selectable
- Models are downloaded from HuggingFace into a shared on-device cache
- Inference is serialized through an actor lock with **pre-emption** — your chat tap cancels in-progress background generation so the UI feels instant

## AIHA — on-device chat

- Ask anything in natural language: *"how did I sleep this week?"*, *"what helps my energy?"*, *"why did I feel tired yesterday?"*
- Answers are grounded in **your** data via on-device retrieval (RAG over your local CoreData entries)
- Charts inline in chat where helpful (e.g. weekly sleep bar chart)
- **Pro Mode** — larger context window, deeper analysis, longer time ranges
- Pre-warming — the model session starts loading the moment you open the chat tab, so the first response feels immediate
- Chat history persisted locally as JSON — no server, no account
- Conversation never leaves the device

## Dashboard

- **Unified view** — sleep (with CSQI), mood, energy, steps, well-being, all on one screen
- **Card-based** with user-configurable order and visibility
- **Sparklines** and trend chips on every card
- **24-hour circadian view** — sleep, awake, and active periods around the clock
- **Sleep schedule consistency** — bedtime/wake-time variance with statistical bands
- **Daily Well-Being score** — weighted across mood, stress, energy, and physical health

## Apple Watch app

- **Quick logging** — mood, stress, energy, drinks from the wrist
- **Well-being view** — current score and recent trend
- **Offline queue** — entries logged while disconnected sync the moment you reconnect
- **WatchConnectivity** sync (not CloudKit, so it works without internet)
- **Complications** for the well-being score on every watch face

## Home-screen widgets

Three widgets, all sizes where applicable:

- **Well-Being** — current score and trend
- **Weather & Mood** — daily mood logged against the day's weather (uses Apple WeatherKit)
- **AI Insight** — your most recent Layer 2 or Layer 3 insight, refreshed automatically

All widgets read from a shared App Group container — they never touch HealthKit or CoreData directly, and refresh is event-driven from the main app.

## Notifications

- **Insight notifications** — new Layer 2/3 findings with deep links to the relevant detail
- **Well-being reminder** — daily prompt to log mood (configurable time, default 18:00)
- **Log reminders** (Premium) — multiple per-metric reminders configurable through settings
- **Duplicate suppression** — local-only tracker prevents firing the same insight twice

## Privacy and data

- **All correlation analysis runs on-device**
- **All AI inference runs on-device** (Apple Foundation Models or MLX)
- **No server-side health database** — Ahimo does not operate one
- **Optional iCloud backup** — opt-in, never enabled by default. Your data goes to *your* iCloud, with Apple as the controller. Under Advanced Data Protection, the keys are yours.
- **Mandatory crash reporting** via Firebase Crashlytics — never includes health data
- **Optional analytics** — opt-in, anonymized, disable anytime in Settings
- **Three-language support** — English, Deutsch, Español

## Localization

- Single source-of-truth string catalog for the iPhone app, separate catalog for Watch
- EN, DE, ES at parity
- Pluralization handled per-locale

## Architecture highlights

- **MVVM + Services**, with a single `UnifiedDataManager` gateway between view models and data sources
- **CoreData with NSPersistentCloudKitContainer** — automatic migration, persistent history tracking always on
- **Background tasks** — Layer 2 generation every 24h, Watch sync every hour
- **App Group** sharing for widgets
- **Cross-app SharedAI package** — Ahimo and the sister app **Falcata** can share downloaded MLX model weights through a common app group, so a multi-GB model only needs to be downloaded once

## Cross-app data exchange (with Falcata)

- Ahimo writes a versioned JSON snapshot of the last 90 days of user-logged metrics into a shared app group container
- Sister app **Falcata** (sprint training) can read it (read-only) to enrich its own analysis with Ahimo's mood/sleep/recovery context
- Both apps share the same on-device MLX model cache — first download wins, the second app skips the download and mmaps the existing weights
