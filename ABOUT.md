# About Ahimo

**Ahimo** is a privacy-first health app for iPhone, with an Apple Watch companion and a set of home-screen widgets. It is designed for people who already have a lot of biometric data (Apple Watch, HealthKit) and want to actually understand it — to see the real, personal patterns connecting their habits, their feelings, and their bodies.

The core idea is simple: **biometrics alone are not enough**. Your watch knows your heart rate, your steps, and how long you slept. It does not know whether you had two glasses of wine at dinner, whether you felt anxious about a meeting, whether your period started, or whether your back has been hurting all week. Without that context, every "insight" your tracker offers is a guess. With that context, real patterns emerge — and that is what Ahimo is built to surface.

> **Important:** Ahimo is **not a cloud health platform**. Your health data never leaves your iPhone. All correlations, all statistics, and all AI-generated insights are computed on-device, using either Apple's Foundation Models or a local MLX model running on your Neural Engine. The only things Ahimo ever transmits are mandatory crash logs (no health data) and optional usage analytics you can disable in Settings.

## The problem Ahimo solves

The standard health-app experience looks like this:

1. Your watch tracks your sleep and gives you a score.
2. You feel terrible the next day and have no idea why.
3. The app suggests "go to bed earlier," which you already know.

The reason that loop is broken is that **the app is missing half the data**. It cannot see that you had three drinks last night, or that your stress was high all week, or that you are five days into your luteal phase. Without that context, no correlation engine — no matter how sophisticated — can give you anything beyond generic advice.

Ahimo closes the loop by making logging the missing context **fast** (a tap on your watch, a swipe on your phone) and by then doing the analytical work that actually matters: finding the personal correlations between what you logged and what your body did.

## What Ahimo does differently

Ahimo runs a **three-layer insights pipeline** on-device, with each layer answering a different kind of question.

**Layer 1 — Statistical baselines.** When you open a detail view, Ahimo computes the relevant statistics on the fly: 7-day averages, weekly deltas, percentile bands. Nothing is persisted. This is the "what is happening" layer.

**Layer 2 — Correlation analysis.** Thirty purpose-built analyzers run in the background looking for the patterns in your data. Sleep impact, HRV correlations, activity effects, menstrual-cycle phase analysis, mood-energy bidirectional patterns, drinks-on-sleep, and more. Each finding is an explained sentence: *"On days when you skipped drinks, your sleep quality improved by 9 points."* This is the "why is it happening" layer.

**Layer 3 — On-device LLM.** Once a week, an on-device language model (SmolLM3, Qwen3.5, Llama 3.2 via MLX, or Apple Foundation Models) reads your Layer 2 findings and your profile and writes deeper, narrative insights. It is also the engine behind **AIHA**, the chat assistant. This is the "talk to me about it" layer.

The whole pipeline is built around one rule: **data is sacred**. No simulated numbers, no fake "wellness scores" derived from formulas the user can't see, no telemetry on what you log. Every value in Ahimo traces back to HealthKit, your own input, or a clearly disclosed computation.

## AIHA — the on-device assistant

AIHA is your private health AI. You ask questions in plain language:

- *"How did I sleep this week?"*
- *"Why did I feel so tired yesterday?"*
- *"What helps my energy the most?"*
- *"Has my HRV been recovering since I cut back on alcohol?"*

AIHA reads your recent Layer 2 insights, retrieves the relevant entries from your local database, and writes a grounded answer with charts where appropriate. The model runs on your iPhone's Neural Engine. Your conversation never leaves the device. There is no account, no server, no log of what you asked.

A **Pro Mode** unlocks larger context, more detailed analysis, and the ability to query longer time ranges.

## What you can log

Ahimo's strength is in the context you add. Logging is designed to take seconds:

- **Mood** and **energy** (quick taps, on watch or phone)
- **Stress** level
- **Drinks** — alcoholic and caffeinated, with type and time
- **Symptoms** — headache, nausea, soreness, anything you choose to track
- **Intimacy**
- **Menstrual** events and cycle tracking
- **Tags** — free-text or structured labels for anything else

Everything is timestamped with three distinct dates: when you logged it, when it actually happened, and which calendar day it belongs to (so a 1 AM drink belongs to the previous day's data, where it should).

## Built for the Apple ecosystem

Ahimo is iOS-native and uses every Apple platform feature that makes sense for a health app:

- **HealthKit** is the source of truth for sensor data. Ahimo reads sleep, heart rate, HRV, steps, workouts, and menstrual data, and writes back what makes sense.
- **Apple Watch** is the quick-logging surface. Mood, stress, energy, drinks — all loggable from the wrist, with an offline queue that drains the next time the watch reconnects.
- **Home-screen widgets** — three of them: a Well-Being score, a Weather & Mood card, and the latest AI Insight.
- **Watch complications** keep your well-being score one glance away.
- **iCloud** (optional, opt-in) backs up your data via CloudKit, with Apple as the controller.
- **Apple Intelligence** — when you're on a supported device, Ahimo's chat and Layer 3 insights can use Apple's Foundation Models framework, running fully on-device.
- **MLX** — for users on devices without Apple Intelligence, or who want a different model, Ahimo can download and run open MLX models (SmolLM3-3B, Qwen, Llama) entirely locally.

## Who Ahimo is for

**Curious self-trackers.** You have years of Apple Watch data. You want to know what it actually means for *you*, not what generic studies say.

**People navigating sensitive health changes.** Cycle tracking, recovery from illness, alcohol moderation, sleep hygiene experiments. Ahimo's correlations are personal, so the conclusions are too.

**Privacy-first users.** You will not put your menstrual data, your drinking history, or your mental health in someone else's database. Ahimo is built so you don't have to.

**People who want a real AI health conversation.** Not a chatbot trained to upsell you supplements — an AI that knows *your* data, gives you grounded answers, and runs on your phone.

## The philosophy

Health is contextual. Sleep is not just hours; it is hours after a stressful day or after two drinks or two weeks before your period. Ahimo's job is to hold that context for you and to do the math you don't want to do yourself.

It is also built on a strict promise: your data is yours. The AI is real, the insights are personal, and nothing has to leave your device for any of it to work. That is not a marketing claim — it is the architectural constraint the app is built around.

---

Learn more at [ahimo.app](https://ahimo.app). Designed in Switzerland.
