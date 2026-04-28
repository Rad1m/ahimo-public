# Ahimo — Frequently Asked Questions

## The basics

### What is Ahimo?

Ahimo is a privacy-first iOS health app that pairs your Apple Watch and HealthKit data with the personal context only you can provide — mood, stress, energy, drinks, symptoms, menstrual cycle — and uses on-device AI to find the patterns that actually affect your sleep, recovery, and well-being.

### Who is Ahimo for?

People with an Apple Watch who want their biometric data to actually mean something. Self-trackers, anyone navigating sleep or stress changes, people tracking menstrual cycles who want phase-aware insights, and anyone who refuses to upload their health data to a cloud service.

### What platforms does Ahimo run on?

iPhone (primary), Apple Watch (companion), and home-screen widgets. iCloud is supported as an optional backup.

### What languages does Ahimo support?

English, Deutsch, and Español.

### Is Ahimo free?

Ahimo is on the App Store with a free tier and a Premium subscription. Premium unlocks features like Pro Mode for AIHA, log reminders, and deeper analysis. Visit [ahimo.app](https://ahimo.app) for current pricing.

## How it works

### What does Ahimo actually measure?

Ahimo combines two streams of data:

- **From HealthKit** — sleep (with stages), heart rate, HRV, steps, workouts, blood pressure, menstrual data
- **From you** — mood, stress, energy, drinks (alcoholic and caffeinated), symptoms, intimacy, menstrual events, free-text tags

It then runs 30+ correlation analyzers on-device to find the personal patterns between the two.

### What is AIHA?

AIHA is Ahimo's on-device chat assistant. You ask questions in natural language ("how did I sleep this week?", "what helps my energy?") and AIHA gives grounded answers using your own data, with charts where helpful. It runs entirely on your iPhone — no server, no account, no transmission of your data.

### What is CSQI?

CSQI — the **Composite Sleep Quality Index** — is a single 0–100 score Ahimo computes from your sleep duration, schedule consistency, and stage composition. It is the score used in the dashboard and the Well-Being widget.

### What is the "three layer" insight system?

- **Layer 1** is on-the-fly statistics (averages, deltas, percentiles) for detail views.
- **Layer 2** is correlation analysis from 30 background analyzers, refreshed every 24 hours and on demand. These are the "Drinks → Sleep: −12 minutes" insights.
- **Layer 3** is on-device LLM-generated narrative insights, written weekly, that synthesize Layer 2 findings into deeper observations.

### What AI model does AIHA use?

Two options, selectable in Settings:

- **Apple Intelligence** (the Foundation Models framework) on supported devices, running on the Neural Engine
- **MLX** — open models downloaded from HuggingFace and run locally. The default is `SmolLM3-3B-4bit`. `Qwen3.5-4B` and `Llama-3.2-1B` are also supported.

In both cases, inference happens on your iPhone. Your data is never sent to OpenAI, Anthropic, Apple Private Cloud Compute, or any other external service.

### How does Ahimo handle menstrual cycles?

Seven dedicated analyzers correlate cycle phase against sleep, symptoms, HRV, mood, stress, energy, and intimacy. The phase detector reads HealthKit menstrual data and (optionally) your logged events.

### Does Ahimo replace my doctor?

No. Ahimo's insights are informational and not medical advice. If something in your data concerns you, talk to a clinician.

## Setup and devices

### Which iPhones work with Ahimo?

Any modern iPhone that supports the version of iOS Ahimo targets. For on-device AI features, a device with the Neural Engine and enough RAM to run a multi-GB MLX model is recommended; older devices can fall back to lighter models or use Apple Intelligence where available.

### Do I need an Apple Watch?

No, but it makes Ahimo much more useful. The Watch is your quick-logging surface (mood, stress, drinks from the wrist), and it's the source of the high-quality biometric data that drives the correlations.

### Which Apple Watch models work?

Any Apple Watch that can pair with your iPhone and write the relevant HealthKit data. Newer models give you better sleep stage detection and HRV.

### Does Ahimo work without an internet connection?

Yes. All analysis and AI runs on-device. The only things that need internet are the initial MLX model download (one-time, multi-GB) and optional iCloud sync.

## Data and privacy

### Does Ahimo upload my health data?

No. Health data, AIHA conversations, and on-device AI computations stay on your device.

### What does Ahimo send off-device?

- **Mandatory:** anonymous crash reports via Firebase Crashlytics. These never contain health data — only crash stacks, device model, OS version, app version, and a randomly generated install UUID.
- **Optional, opt-in:** anonymous usage analytics via Firebase Analytics, including approximate (city/region) location derived from IP. You can disable this in Settings at any time.

### What about iCloud?

iCloud backup is **optional and opt-in** — never enabled by default. If you turn it on, your data is backed up via Apple's CloudKit infrastructure. Apple is the data controller; Ahimo never sees the contents. Under Advanced Data Protection, the encryption keys are yours alone.

### Where is my data stored?

Locally on your iPhone in CoreData (with hardware-backed iOS encryption), and on the Apple Watch in its own offline queue. Optionally, in your own iCloud account if you enable backup.

### Can I delete my data?

Yes. Delete entries individually, clear categories, or uninstall the app to remove everything local. If iCloud backup was enabled, deleting from one device propagates to your other devices through iCloud.

### Is Ahimo GDPR-compliant?

Yes. Ahimo is operated from Switzerland (Stein, Switzerland) and complies with GDPR and the Swiss FADP. The full privacy policy is available in the app and at [ahimo.app](https://ahimo.app).

### Does Ahimo use my data to train AI models?

No. There is no training pipeline. The MLX models you run are pre-trained open models downloaded from HuggingFace. Apple's Foundation Models are used as-is. Your data is not used to train, fine-tune, or improve any model — Ahimo's or anyone else's.

## AIHA chat

### Can AIHA see my entire history?

AIHA uses on-device retrieval (RAG) over your CoreData entries plus a summary of your recent Layer 2 insights. It pulls in the slice of your data that's relevant to the question you asked. **Pro Mode** widens the context window and time range.

### Does AIHA give medical advice?

No. AIHA is built to describe your data and surface patterns. It does not diagnose, prescribe, or replace a clinician. The system prompt explicitly includes a "not medical advice" disclaimer.

### Why is the first AIHA response slow?

The first time you open the chat tab in a session, the model session has to start loading. Ahimo pre-warms the session the moment the chat tab becomes visible to make this as short as possible. Subsequent responses are much faster.

## Comparison

### How is Ahimo different from Apple Health?

Apple Health is a *store* — it holds your biometric data. Ahimo is an *analyst* — it adds your contextual logging (mood, stress, drinks, symptoms) to that biometric data and runs correlation analysis to find personal patterns. Ahimo also adds the on-device AI assistant Apple Health does not have.

### How is Ahimo different from Bevel, Welltory, Oura, Whoop?

Most of those apps either (a) require their own hardware or (b) send your data to a cloud for processing. Ahimo uses your existing Apple Watch and runs everything on-device.

### How is Ahimo different from a generic AI chatbot like ChatGPT?

ChatGPT does not know your health data, and uploading it to a cloud LLM raises real privacy questions. AIHA runs on your phone, knows your data, and never transmits it.

## Cross-app

### What is the relationship to Falcata?

Falcata is a sister app — a sprint training app for track and field athletes — built by the same team. The two apps share an on-device MLX model cache (so you only download a multi-GB model once across both apps) and Ahimo can optionally write a snapshot of recent user-logged metrics into a shared container for Falcata to read. Neither app uploads data to do this; the sharing happens entirely through a local app group on your device.

---

Have a question that isn't here? Visit [ahimo.app](https://ahimo.app).
