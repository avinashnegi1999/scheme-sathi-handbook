# Architecture

---

## The shape in one diagram

```
        Telegram
            │
            ▼
   ┌─────────────────┐
   │  channels/      │   translation only — holds no logic
   │  telegram.py    │   keyboards, callbacks, message ids
   └────────┬────────┘
            │  Reply / ChannelMessage
            ▼
   ┌─────────────────┐
   │  conversation/  │   the intake state machine
   │  flow.py        │   asks, records, never decides
   └────────┬────────┘
            │  Profile
            ▼
   ┌─────────────────┐
   │  rules/         │   ◀── NO LLM MAY BE IMPORTED HERE
   │  engine.py      │       pure functions, deterministic
   └────────┬────────┘
            │  Result (ELIGIBLE | INELIGIBLE | UNKNOWN)
            ▼
   ┌─────────────────┐
   │  render/        │   turns a Result into Hindi or English
   │  templates.py   │   llm.py may rephrase — never decide
   └────────┬────────┘
            │
            ├──▶ pack/      the one-page sheet to carry to the centre
            └──▶ metrics/   coarse bands only, random session id
```

---

## The boundary that defines the project

There is exactly one architectural rule that matters more than the rest:

> **`sathi/rules/` decides eligibility and may not import a language model.**

Everything above that line is conversation. Everything below is arithmetic.
The data crossing the boundary is a `Profile` going down and a `Result` coming
back up — both plain, both inspectable, neither containing prose.

This is why the project can claim determinism honestly. It isn't a policy
someone has to remember during code review; it's an import that isn't there.

---

## Module map

| Module | Responsibility | Decides eligibility? |
|---|---|---|
| `core/profile.py` | The only data the engine reads | no — it's the input |
| `core/schemes.py` | Load and validate `data/schemes/*.toml` | no — it gates servability |
| `core/content.py` | Occupations, states, UI strings | no |
| `rules/operators.py` | The seven comparison operators a scheme may use | **yes** |
| `rules/engine.py` | The only place eligibility is decided | **yes** |
| `conversation/consent.py` | Consent, asked first, before anything is recorded | no |
| `conversation/flow.py` | Intake state machine — asks, records, hands off | no |
| `channels/base.py` | Channel-agnostic message types | no |
| `channels/telegram.py` | Telegram adapter — thin, holds no logic | no |
| `render/templates.py` | Results into Hindi a worker can hear | no |
| `render/llm.py` | Optional rephrasing, strictly outside the engine | no |
| `render/audio.py` | Hindi audio notes — optional, pluggable | no |
| `pack/checklist.py` | Which documents are needed, which are missing | no |
| `pack/pack.py` | The one-page application pack | no |
| `metrics/events.py` | The **only** writer to the event database | no |
| `metrics/report.py` | SQLite in, one self-contained HTML file out | no |

Two modules decide anything. That is the point.

---

## Why the channel is separable

`channels/base.py` defines `Reply` and `ChannelMessage`. Telegram is one
implementation; WhatsApp is designed for and unbuilt.

The reason is not architectural purity — it's a schedule risk. WhatsApp
requires Meta business verification, which can take weeks and can fail. Making
the channel swappable meant a verification delay could never block a deploy
date. The abstraction exists to de-risk a calendar, which is a better
justification than "it's cleaner".

> **Honest caveat:** since only Telegram is implemented, "channel-agnostic" is
> argued rather than demonstrated. Say so if asked.

---

## Zero dependencies

`dependencies = []`. Stdlib only, Python 3.11+.

The multipart file upload, the Telegram HTTP client, the TOML loading, the
SQLite access, and the rule engine are all hand-written against documented APIs.
`python3 check.py` runs everything with nothing to install.

This was a deliberate constraint, and it paid off in a way that wasn't planned:
deploying to a fresh VM needed no package step, so the difference between
"works on my laptop" and "works in production" was one `rsync`.

---

Next: [04-the-rule-engine.md](04-the-rule-engine.md)
