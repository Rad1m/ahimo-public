# On-Device SLM Investigation: Prompt Iteration, Hallucinations, and Agentic Tool Use

**Author:** Radim Simanek
**Date:** 2026-05-01

How do 3B-class language models running on-device on Apple Silicon
actually behave when you ask them to reason about HealthKit-style data?
This is a write-up of a focused investigation: four models, two real
health-data tasks, and the failure modes that matter when the answers
the model produces are going to be shown to a person trying to
understand their own body.

The motivation is simple. Health apps that compute correlations and
generate insights *on-device* are bound by what a small quantised model
can do reliably without a server in the loop. "Reliably" is doing a lot
of work in that sentence. A 3B model that produces a fluent, confident,
*wrong* answer about whether alcohol improved your sleep is worse than
no answer at all. So the question we cared about was not "which model
is smartest" but "which models fail in ways we can detect and contain,
and on what kind of HealthKit-style prompt do they fail?"

## Models tested

All four are 4-bit MLX quantisations runnable on a current Apple
Silicon Mac or iPhone:

- **SmolLM3-3B** (~1.7 GB) — Hugging Face's 3B model, smallest
- **Qwen3.5-4B** (~2.9 GB) — Alibaba's 4B model
- **Llama-3.2-3B-Instruct** (~1.9 GB) — Meta
- **Phi-3.5-mini-instruct** (~2.2 GB) — Microsoft

These are the realistic candidates today for an on-device health app
that wants to ship a chat experience without sending data to a server.
We tested them via `mlx-lm` on Apple Silicon, holding the model in
memory across iterations so prompt-engineering loops could run in
seconds rather than the minutes it takes to rebuild a native app.

## What we tested them on

Two task types, both grounded in HealthKit-style data shapes:

**1. Correlation chat prompts.** "Does my mood and sleep correlate?"
"Is alcohol affecting my HRV?" — the kind of question a user asks an
in-app assistant. Two fixtures:

- `mood_sleep_negative_week` — n=7 paired daily values, ordinal mood
  on a 1–5 scale, sleep duration in hours. Qualitative answer territory
  (n=7 is too small for a meaningful coefficient).
- `alcohol_hrv_two_weeks` — n=14 paired daily values, alcohol units,
  HRV in ms. Strong negative monotonic relationship (Spearman ρ ≈
  −0.85). Big enough that a coefficient is appropriate, but the model
  must not invent per-drink dose-response numbers that aren't in the
  data.

**2. Agentic logging — tool calls.** "Log my mood: very anxious." "I
had three glasses of wine." The model has to emit a structured
tool-call (JSON) selecting from a closed set of logging tools (mood,
stress, energy, drinks, symptoms, intimacy, plus `ask` and `refuse`
safety valves) with a typed argument schema. 20 fixtures: 8 easy
(unambiguous), 6 medium (typos, ambiguous severity), 6 hard
(adversarial — mood with no value, multiple metrics in one sentence,
out-of-domain).

## Findings: correlation chat

### Hallucination patterns that actually matter for health data

The literal-coefficient probe — does the model invent an `r=0.7` it
wasn't given — was passed by every model on every fixture. That is the
*easy* failure to catch. The hard ones, the ones that show up only
when you read the outputs:

**Dose-response fabrication.** SmolLM3-3B, on the alcohol/HRV fixture,
produced:

> "...each additional unit of alcohol associated with a ~2 ms decrease
> in HRV, as seen from the paired data."

That number is fabricated. The fixture supplied a Spearman label and
the paired daily rows; it did not supply a per-drink slope. A 3B model
sees a strong monotonic relationship and fluently invents the missing
number. This is the failure mode that matters most for health apps —
because the number sounds authoritative, it is hard to refute, and a
user will absolutely change their behaviour because of it.

**Tokenizer-stop bugs that look like hallucination.** Phi-3.5-mini's
4-bit MLX quant leaks past its `<|end|>` token, then *invents an answer
to its own follow-up question* with fabricated sleep numbers. This is
not a model-quality problem — it is a tokenizer/quant configuration
problem — but it is indistinguishable from hallucination at the user-
facing layer. Until the chat-template stop tokens are right, Phi-3.5
in this quant is not deployable in a chat surface.

**Over-eager safety rules.** Llama-3.2-3B opens its n=7 mood/sleep
answer with *"I don't have that data yet"* and then, in the next
sentence, **answers using the data**. The system prompt told it to say
this when data is missing; the model conflates "no rho was computed"
(because n=7 is too small) with "no data exists". Soft system-prompt
rules need to be scoped tightly when the consumer is a small model.

### Trait-match scores on the two fixtures

Each output was scored on six binary traits: cites a specific date,
cites a specific value, no invented coefficient, no invented dose-
response, follows response-format rules, stays within length budget.

| Model | mood/sleep n=7 | alcohol/HRV n=14 |
|---|---|---|
| **Qwen3.5-4B** | 5/6 | 5/6 |
| SmolLM3-3B | 5/6 | 5/6 (passes regex; *fails* on dose-response read) |
| Phi-3.5-mini | 4/6 | 4/6 (plus tokenizer leak) |
| Llama-3.2-3B | 4/6 | 3/6 |

The headline scores are close. Reading the outputs is what separates
them. Qwen3.5-4B is clearly the strongest on health-data reasoning:
cites specific dated values, follows response-format rules, uses the
qualitative aggregates supplied for small N rather than fabricating
coefficients. SmolLM3-3B is competitive on safety at small N but
hallucinates a dose-response at n=14 when the data invites it.
Phi-3.5-mini and Llama-3.2-3B were not viable for this use case in
their current 4-bit MLX form.

### What prompt engineering closed and what it didn't

Three rule changes, evaluated in the harness across both models and
both fixtures:

- **Scope "missing data" rules to the *requested* metric.** Stops
  small models from misfiring when one part of the prompt context is
  empty.
- **Explicitly forbid per-unit / dose-response phrasing**, with
  concrete examples in the prompt: "do not say things like '~2 ms per
  drink' or '+5 points per hour'".
- **Make date+value citation mandatory with an example format
  string.** Models routinely ignored *"Mention specific values and
  dates"* but followed *"You MUST cite … Format: 'Thursday: 8h 10m
  sleep, 5/5 mood'"*.

| version | Qwen mood | Qwen alcohol | Smol mood | Smol alcohol | Total |
|---|---|---|---|---|---|
| baseline | 5/6 | 5/6 | 5/6 | 5/6 | 20/24 |
| **+ three rules** | **6/6** | **6/6** | **5/6** | **5/6** | **22/24** |
| + harder anti-dose-response | 6/6 | 5/6 | 4/6 | 6/6 | 21/24 |

The "+ three rules" prompt is the clean improvement: Qwen reaches the
ceiling, SmolLM3 stays where it was. Pushing the dose-response rule
further regresses Qwen's citation behaviour and makes SmolLM3 over-
cautious without fully fixing the dose-response framing — a useful
reminder that prompt rules trade against each other on small models in
ways they don't on frontier ones.

**The residual limitation, accepted.** SmolLM3 still produces a soft
dose-response framing on the alcohol/HRV fixture (*"most significant
drop at 5 units, leading to a 12 ms decrease"*) that the regex scorer
can't catch and the prompt couldn't fully suppress. The cleanest
mitigations are (a) a post-processor that detects and rewrites these
phrases, (b) routing high-stakes correlation prompts to the larger
model, or (c) an LLM-as-judge eval rubric — none of them free.

## Findings: agentic tool calls for health logging

A different shape of task. The model has to read free-form input ("I
had three glasses of wine, the second one was huge") and emit a JSON
tool call against a closed schema:

```
log_mood       { mood_types: [<60 enums>], intensity: 1-5 }
log_stress     { level: 1-5 }
log_energy     { level: 1-5 }
log_drink      { drink_type: <12 enums>, amount: number }
log_symptom    { symptoms: [<50 enums>], intensity: 1-5,
                 duration: minutes|hours|days|weeks|null,
                 onset: sudden|gradual|null }
log_intimacy   { subtype: emotional|physical|both, protection: bool|null }
ask            { question: string, missing: [arg names] }
refuse         { reason: string }
```

The `ask` and `refuse` tools are deliberate safety valves. If the
model can't make sense of the user's input, it must say so — never
guess.

### Smoke-test results across 20 fixtures

| | easy (8) | medium (6) | hard (6) | total |
|---|---|---|---|---|
| **Qwen3.5-4B** | 7/8 | 5/6 | 6/6 | **18/20 (90%)** |
| SmolLM3-3B | 7/8 | 2/6 | 5/6 | 14/20 (70%) |

The gap is wider here than on the chat task. Free-form-to-structured
extraction with a closed enum vocabulary is a place where the extra
parameter count of the 4B model is paying for itself.

### Failure modes

**Safe failures** (confirmation UI catches them):
- Misreads typo *"Said"* as `log_stress` when *"sad"* was intended →
  routes to mood
- Maps *"Wiped out"* to `log_stress` when `log_mood: tired` is the
  correct mapping — the semantic axes overlap
- Off-by-one intensity inference from severity adjectives ("very" → 4
  vs 5)

**Unsafe failure (one):**
- *"Log my mood"* with no value: SmolLM3 defaults to
  `mood_types: ["happy"]` despite the system prompt's explicit "NEVER
  default — use `ask`" rule. Qwen correctly emits `ask`. Without a
  user-facing confirmation step, this would commit a wrong mood.

The mitigation is structural rather than a prompt fix: a code-level
guard that detects "Log my X" patterns with no value and forces the
`ask` path regardless of model output. The lesson generalises — when a
small model has a high-confidence default that's wrong in adversarial
inputs, you cannot prompt your way out of it. You contain it in code.

### Coercion the validator has to do

Both models routinely emit JSON like `{"level": "3"}` instead of
`{"level": 3}`, or `{"protection": "true"}` instead of `true`. A
validator that treats these as failures will reject responses that any
human would consider correct. Building lenient string→number and
string→bool coercion into the validation layer was the single biggest
unlock for getting score numbers to reflect actual capability rather
than serialisation pedantry. This applies on the iOS side too: any
JSON-to-typed-struct boundary should coerce, not just decode.

## What this means for shipping

Three things hold up across both task types:

1. **Qwen3.5-4B is the strongest answer quality on health-data
   prompts** by a clear margin once you read past the regex scorers.
   The cost is ~1.2 GB more disk, slower load, more memory pressure on
   older devices.
2. **SmolLM3-3B is competitive on bounded prompts** (small N, simple
   intent) and viable as a default if you can either route the
   high-stakes prompts to a stronger model or accept a documented
   residual hallucination rate on dose-response framing.
3. **Phi-3.5-mini and Llama-3.2-3B in their current 4-bit MLX form
   were not viable** for this use case. Phi has a tokenizer-stop bug
   that produces compounding hallucination; Llama misfires conditional
   safety rules in ways that erode trust.

The harness itself — Python, mlx-lm, models loaded once, fixtures in
JSON, scoring deterministic — turned out to be the most valuable
artefact of the investigation. Iteration cycles on a system prompt
collapsed from 3–5 minutes per round (rebuild + simulator deploy) to
1–2 seconds per round. That is the difference between guessing and
testing.

## Open questions worth chasing

- **Can a stronger system prompt close the SmolLM3 gap on
  alcohol-style prompts?** A one-shot example of the desired
  structure, not just rules, would be the next thing to try.
- **LLM-as-judge for the failures regex can't see.** The dose-response
  failure mode is the obvious target — a small judge model evaluating
  the chat output against a "no fabricated per-unit numbers" rubric
  would catch what regex can't.
- **Constrained decoding via Apple Foundation Models** for the
  agentic-logging path. The harness only covers the MLX path; a
  schema-constrained decoder side-by-side comparison is the natural
  next step.

The investigation was scoped to be useful, not exhaustive. The output
that mattered most was a calibrated sense of where these models break
on real health data — and the harness to keep that sense calibrated as
the models keep improving.
