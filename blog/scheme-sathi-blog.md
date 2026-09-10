# I built a welfare bot that refuses to answer

## Scheme Sathi — how I picked the problem, what I got wrong, and what it was actually like building it with Claude Code

*Draft 1. Not published yet.*

---

## The thing I keep coming back to

There is a number in the Government of India's own 2025 labour force survey that decided the entire architecture of this project before I wrote a line of code.

**A casual labourer in India earns ₹455 a day if male, ₹315 if female.**

That is not a rhetorical figure I picked up somewhere. It is from the Periodic Labour Force Survey Annual Report 2025, published by the National Statistical Office. I can point at the PDF.

Here is why it matters. Imagine you are that worker. Somebody tells you there is a government scheme you qualify for. To claim it, you have to go to a Common Service Centre — which means not working that day, and paying bus fare both ways. You go. You stand in line. The operator looks at your papers and says no, you don't qualify for that.

You have just paid ₹455 for a wrong answer. And in most cases, you do not come back for a second attempt.

So when I sat down to build something that tells a worker whether they qualify for a scheme, "should the bot be careful?" was not a question of taste. It was arithmetic. A wrong **yes** costs a day's wages. A wrong **no** costs an entitlement they may have been owed for years, and they will never know to ask again.

**Those two errors are not the same size and they are not in the same unit.** One costs a day. The other costs years. Most systems treat false positives and false negatives as a dial you tune. Here they are different currencies.

Everything below follows from that.

---

## Part 1 — How I got to this problem

### I built a different project first, and killed it

Scheme Sathi is my second project for the Code for a Billion hackathon. The first was **Saans**, a personal air-exposure agent on Telegram. Around 3,500 lines, 13 modules, zero third-party dependencies, all checks passing, a live bot, a public repo under Apache-2.0.

It is finished code. I parked it on 30 August 2026.

**How I picked that track in the first place**, which is a slightly embarrassing thing to admit: not by what I found interesting. 50% of the hackathon score is a written problem statement judged on the size and severity of the problem, and it has to be cited. So I ranked the tracks by headcount and death toll that I could actually source. Clean Air — 1.4 billion people exposed, 1.67 million deaths a year — beat Tech Employment's 1.5 million graduates a year by roughly 18 points before a single line of code existed. The track was also nearly empty, and judging fell during the North India smog peak, so the impact data would collect itself.

That is a defensible way to pick. I still stand by it.

**Why I parked it anyway** is the part worth being precise about, because I lied to myself first.

The reason I gave myself at the time was that I had discovered the idea already existed in another app. That felt bad, so I stopped. But novelty is not a judging criterion in this hackathon. Nowhere in the rules does it say the idea has to be new.

The real reason is that Saans was **never deployed and never put in front of a single user.** 25% of the score is deployment and impact data, and ties break on impact first. I had zero users.

> Losing on novelty was a feeling. Losing on *"25% is deployment and impact, and you have none"* is a score.

I find that distinction genuinely useful now. Being able to say "I told myself one reason and the real one was different, here's how I noticed" is a more honest thing to have on a portfolio than a project that never got killed.

**What Saans taught me technically**, all of which is load-bearing in Scheme Sathi:

- **"Compute, then narrate."** Saans is a health product. A hallucinated µg/m³ figure is not cosmetic. So every number was computed in Python and the message layer only rendered facts it had been handed. That rule is the direct ancestor of "the rule engine may not import a language model", which is the single most important decision in this codebase.
- **Zero dependencies, discovered rather than decided.** I had planned to use FastAPI, python-telegram-bot and APScheduler. Each turned out to be about 80 lines of `urllib` for what I actually needed. All three got dropped. Scheme Sathi started from that position instead of arriving at it.
- **Constraints that veto and explain, never a blended score.** Saans lets heat and roadside traffic *veto* an hour and say why, instead of folding everything into one opaque number. Scheme Sathi's three-valued verdicts with a reason line per criterion are the same idea.
- **I found Saans's two worst bugs by using it myself.** It recommended a 45-minute run at 10am in 36°C heat, because the cleanest hour of the day is very often the hottest — same physics, and the model had no idea. And it forecast coarse-grid ambient air, while most people in India run on roads, where kerbside exposure at commute peaks is materially worse. Both were obvious the second a real person read the output. Neither was visible from the test suite.

That last one happened again in Scheme Sathi, almost word for word. I'll get to it.

### The problem I moved to

India's unorganised sector is **43.99 crore workers** — the Economic Survey 2021-22 figure, quoted by the Ministry of Labour & Employment in a Lok Sabha reply. The last full survey to split the workforce found **82.7% of it outside the organised sector**: 39.14 crore of 47.41 crore employed persons, from NSSO 2011-12.

That 2011-12 date is old and I say so in the README rather than hiding it, because the PLFS series that replaced NSSO does not publish the same organised/unorganised split. There is no newer number to quote. Saying "the latest data on this is from 2011-12 and here is why" is better than quietly using a figure that sounds current.

Construction labourers, domestic workers, drivers, street vendors, farm labour, shop staff. Roughly 90% of the workforce. Typically limited literacy, a cheap Android phone, patchy 4G.

The welfare infrastructure **already exists.** **31.48 crore** unorganised workers were registered on e-Shram as on 26 January 2026, with 14 central schemes integrated into it — PMSBY, PMJJBY, PM-SVANidhi, AB-PMJAY, PM-KISAN, ONORC and others.

So the gap is not eligibility. These workers are already entitled. The blockers are:

1. They do not know which schemes exist, or which apply to them.
2. The rules are scattered, in English, in bureaucratic language.
3. A failed trip to a CSC costs a day's wages, so the second attempt often never happens.

A worker legally entitled to a pension or an accident cover simply never claims it.

### "But myScheme.gov.in already exists"

It does. It is the government's own scheme directory, it is comprehensive, and it is well built. UMANG does eligibility matching too. I checked both before building anything, and I think checking is the actual difference between solving a problem and noticing one.

Scheme Sathi does not replace them. It is **last-mile delivery on top of them.**

> myScheme is an English web form. It assumes literacy, a browser, and a user who knows what "land holding in hectares" means. Scheme Sathi is a conversation on a ₹6,000 phone that ends in a filled checklist and an address to walk to.

I say this line out loud whenever anyone asks, because "why not just use the government site" is the first question a judge or an interviewer should ask, and not having an answer would be fatal.

---

## Part 2 — The four decisions, made before any code

These were settled on day one and never revisited. Each one cost something, and each one is the reason a whole category of bug never happened.

### 1. The language model may not touch an eligibility decision

`sathi/rules/` cannot import a language model. Structurally, not by convention.

The model is allowed to: run conversation flow, map free text like "I lay bricks" to a job category (always confirmed back to the worker before anything is recorded), and rephrase Hindi that a human wrote. It never sees a threshold. It never produces a rupee figure. It never produces a verdict.

The shorthand is **compute, then narrate.**

> **Consequence:** the same profile always produces the same answer, and I can point at the exact line of the rule that produced it. A model that is right 95% of the time is wrong once in twenty. For someone spending a day's wages on the answer, that is not a rounding error.

The stronger version of this claim: **`LLM_API_KEY` unset is a tested configuration, not a degraded one.** Every eligibility result is byte-identical with or without it. `tests/test_rules.py` asserts the import graph of `sathi.rules` stays clean, so a stray import fails the build rather than embarrassing me in a demo.

That is the only honest way to prove the model isn't load-bearing.

### 2. An unresearched value is the string `"TODO"`, never `0`

Including for numeric fields:

```toml
annual_value_inr = "TODO"
```

A string, in a numeric field, on purpose.

> **Consequence:** a zero looks researched. No validator can tell "this scheme is worth ₹0" apart from "nobody has looked this up yet". A string breaks loudly the moment anything tries to do arithmetic on it — which is exactly the behaviour I want.

`None` would have type-checked in Python and slipped silently through arithmetic later. `"TODO"` cannot.

The general principle, which I think is worth stealing for any project: **make the unfinished state un-representable as a valid value.** A sentinel that type-checks is a bug waiting for a quiet afternoon.

### 3. Verdicts are three-valued

`ELIGIBLE`, `INELIGIBLE`, `UNKNOWN`.

`UNKNOWN` is returned when a profile field is missing, *or* the scheme file still carries a `"TODO"` that the rule needs, *or* no human has signed the file off. A gap is never filled with a default.

> **Consequence:** the system can say "I have not checked this" — the one thing a confidently wrong answer cannot do. And `UNKNOWN` is not a dead end. It ships with a specific question the worker can ask at the centre, so they still leave with something usable.

### 4. The profile has no name, phone, or Aadhaar field

Not "we don't store it". The fields do not exist in the dataclass.

```
Profile: state, age, occupation, income_band, land_holding_band, family_size,
         has_bank_account, is_income_tax_payer, is_epfo_or_esic_member,
         is_nps_member, known_schemes
```

> **Consequence:** a field that does not exist cannot be leaked by a future bug, a careless log line, or a database dump.
>
> Prose in a privacy policy is a promise. A missing column is a guarantee.

Notice what the bands are doing too: `income_band`, not income. `family_size`, not household composition. Even the data I *do* collect is deliberately coarse.

### The pattern across all four

Each decision moves a safety property from something that has to be **remembered** to something that is **structurally true**. The model can't reach the rules because of an import boundary. A stub can't be mistaken for data because of its type. A worker's name can't leak because there is no field for it.

Discipline is fragile. Structure is not.

---

## Part 3 — How it actually works

### The shape

```
        Telegram / WhatsApp
                │
                ▼
       ┌─────────────────┐
       │  channels/      │   translation only — holds no logic
       │  telegram.py    │   keyboards, callbacks, message ids
       │  whatsapp.py    │
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

There is exactly one architectural rule that matters more than the rest:

> **`sathi/rules/` decides eligibility and may not import a language model.**

Everything above that line is conversation. Everything below it is arithmetic. The only things crossing the boundary are a `Profile` going down and a `Result` coming back up — both plain, both inspectable, neither containing prose.

This is why I can claim determinism honestly. It isn't a policy someone has to remember during code review. It's an import that isn't there.

### The module map

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
| `channels/whatsapp.py` | WhatsApp adapter — same conversation, different wire | no |
| `channels/router.py` | Session routing shared by both channels | no |
| `render/templates.py` | Results into Hindi a worker can hear | no |
| `render/llm.py` | Optional rephrasing, strictly outside the engine | no |
| `render/audio.py` | Hindi audio notes — optional, pluggable | no |
| `pack/checklist.py` | Which documents are needed, which are missing | no |
| `pack/pack.py` | The one-page application pack | no |
| `metrics/events.py` | The **only** writer to the event database | no |
| `metrics/report.py` | SQLite in, one self-contained HTML file out | no |
| `review.py` | The only supported way a name reaches `verified_by` | no |
| `sources.py` | Re-reads official pages, watches for changed numbers | no |

Two modules out of twenty decide anything. That is the point.

### The rule engine

`evaluate(profile, scheme) -> Result`. A pure function. One worker, one scheme, one verdict. No I/O, no clock, no randomness, no network. The same inputs always produce the same output.

Precedence, in order:

| # | Condition | Verdict |
|---|---|---|
| 1 | Scheme not servable | `UNKNOWN` — always, no further checking |
| 2 | Any exclusion definitely hits | `INELIGIBLE` |
| 3 | Any criterion definitely fails | `INELIGIBLE` |
| 4 | Anything undecidable | `UNKNOWN` |
| 5 | Otherwise | `ELIGIBLE` |

**Why that order.** A definite NO outranks a maybe, so rules 2 and 3 sit above 4 — if a worker is definitely excluded, some separate missing field doesn't make that uncertain again.

Rule 1 sits above everything because a scheme nobody has finished must not produce a verdict *at all* — not even a negative one.

### The seven operators

A scheme file may use only these:

| Operator | Meaning | Note |
|---|---|---|
| `exists` | is this field answered at all | the only one decidable with no answer |
| `between` | `[low, high]` | **inclusive at both ends** |
| `gte` / `lte` | numeric threshold | rejects bools — see below |
| `in` / `not_in` | membership in a list | |
| `eq` | exact match | no coercion: `"yes" != True`, `18 != "18"` |

A scheme file cannot express arbitrary logic. It cannot call code, branch, or compute. That is deliberate, and it buys two things: a scheme file is **data reviewable by someone who does not program**, and the set of things a badly written file can do is bounded.

Three details worth knowing:

**`apply()` returns `True`, `False`, or `None`.** `None` means "cannot decide from what we have", and it's what propagates into precedence rule 4.

**There are two ways to get `None`,** both checked before anything else: the expected value is still a `"TODO"` stub, or the worker's answer is missing. `exists` is the exception, checked first, because for that operator absence *is* the answer.

**`bool` is a subclass of `int` in Python.** So `True` would silently compare as `1` against an age band. `_num()` rejects bools explicitly:

```python
if isinstance(v, bool) or not isinstance(v, (int, float)):
    raise OperatorError(f"{where}: expected a number, got {v!r}")
```

That **raises** rather than returning `None`, because it is a scheme-file bug, not missing data. The distinction runs through the whole engine: uncertainty is answered with `UNKNOWN`; malformed input is an error that gets reported and never hidden.

Three-valued logic in practice:

```
criterion: age between [18, 40]

profile.age = 30    →  True   → passes
profile.age = 55    →  False  → INELIGIBLE
profile.age = None  →  None   → UNKNOWN, with "we need your age" attached
```

The third row is the whole design. Most systems would default a missing age to 0, or to "fails", or would refuse to run. This one carries the uncertainty all the way out to the worker and turns it into a question they can ask.

### Scheme files are TOML, not Python

The person who understands welfare policy should not need to understand code.

```toml
code          = "PM_SYM"
name_hi       = "प्रधानमंत्री श्रम योगी मानधन"
authority     = "Ministry of Labour and Employment"
official_url  = "https://maandhan.in/..."
verified_on   = "2026-09-10"
verified_by   = "Avinash Negi, checked 2026-09-10"

[benefit]
annual_value_inr = 36000
value_basis      = "annual_payout"

[[criteria]]
field      = "age"
op         = "between"
value      = [18, 40]
source_url = "https://maandhan.in/scheme-details#eligibility"
pass_hi    = "..."
fail_hi    = "..."
```

Every value carries a **deep-linked `source_url`** — the specific page or anchor that proves it, never the site root — plus a `verified_on` date. A reviewer can audit any single rule in about thirty seconds without opening the code.

A scheme file can be in one of three states, and separating them properly is the most useful idea in the whole data model:

| State | Meaning | Effect |
|---|---|---|
| **STRUCTURAL** | unknown key, bad operator, unknown `Profile` field | `SchemeError` — the app refuses to start |
| **UNRESEARCHED** | a `"TODO"` remains | loads, flagged in `Scheme.stubs` |
| **UNSIGNED** | `verified_by` still says `PENDING HUMAN VERIFICATION` | loads, `is_human_verified` is False |

A structural problem is a **programmer's** bug and should crash loudly at startup. An unresearched or unsigned file is a **process** state — it should load, be visible, and simply refuse to serve verdicts.

Conflating those two was the worst bug in this project's history. I'll come to it.

Adding a scheme needs no Python at all: copy `_TEMPLATE.toml`, fill it from official sources, cite every value. The repo's contribution rule is blunt about it:

> **Contributions without a `source_url` and `verified_on` per value are closed.**

That is not bureaucracy. An uncited threshold is indistinguishable from a guessed one, and a guess costs a worker a day's wages.

### The conversation layer

| | `conversation/flow.py` | `channels/telegram.py` |
|---|---|---|
| Knows about | states, questions, answers | message ids, keyboards, callbacks |
| Does not know about | Telegram | eligibility, or what a scheme is |

`Conversation.handle()` is four lines and dispatches entirely by name:

```python
def handle(self, answer: str) -> list[Reply]:
    answer = (answer or "").strip()
    handler = getattr(self, f"_on_{self.state.value}", None)
    if handler is None:
        return [Reply(text=self._s("errors.generic"), end=True)]
    return handler(answer)
```

The happy path, in order:

```
LANGUAGE → CONSENT → STATE → AGE → OCCUPATION → INCOME → LAND
   → FAMILY → BANK → TAX → [TAX_CONFIRM] → EPFO_ESIC → NPS
   → KNOWN_SCHEMES → DOCUMENTS → PACK → DONE
```

Three-valued logic reaches this layer too. The income tax question offers **Yes / No / Don't know**, and `Don't know` is not stored as `False` — it leaves the field unset, so any scheme excluding tax payers returns `UNKNOWN` with "ask this at the centre" attached.

Same idea as the engine, one layer up: **a person who does not know is not the same as a person who said no.**

### Zero dependencies

`dependencies = []`. Standard library only, Python 3.11+.

The multipart file upload, the Telegram HTTP client, the WhatsApp webhook server, the TOML loading, the SQLite access and the rule engine are all hand-written against documented APIs. `tomllib`, `sqlite3`, `urllib`, `dataclasses` and `unicodedata` cover everything this project does.

This was not a purity exercise, and I did not decide it up front on this project — I arrived at it on Saans, where I planned to use three libraries and found each was about 80 lines of `urllib` for what I actually needed.

The payoff turned out bigger than I expected: **deploying to a fresh VM needed no package step**, so the distance between "works on my laptop" and "works in production" was one `rsync`. There is no `pip install` that can fail on deploy day, and a container that installs nothing cannot fail to install.

---

## Part 4 — Building it with Claude Code

This is the section I actually want to write, because most "I built X with AI" posts either oversell it or get defensive about it, and neither is useful.

### The setup

Windows and Ubuntu at different points, VS Code, Claude Code CLI as the main assistant, and ChatGPT brought in once late in the project specifically as an outside reviewer. Python 3.11 locally, 3.12 on the VM. Git for everything. 79 commits between 31 August and 11 September 2026.

**The single most useful habit was writing documentation as working memory, not as a deliverable.**

The repo has a `docs/` folder with twenty-odd files in it. `BUILD_LOG.md` is 553 lines of what broke and what I got wrong, unedited. `LESSONS.md` is the short version. `VERIFICATION.md` carries the open questions. `SOURCE_REVIEW_2026-09-09.md` records a line-by-line comparison of every shipped value against the live official page.

Those exist because a CLI agent has no memory between sessions, and neither, honestly, do I after four days. Writing the decision down was the only way the reasoning survived. And the second-order effect was better than the first: once the wrong turns were written down, I stopped repeating them.

The rule I ended up with: **if I had to think about it for more than ten minutes, it gets a paragraph in `docs/` before I move on.** Not a tidy paragraph. The wrong version stays in, because the correction is the interesting part.

### What worked

**Plan before edit, every time.** For anything touching more than one file, I had the assistant read the code and produce a plan first, then approve or change the plan, then let it write. Letting a model start editing from a one-line prompt is how you end up with a diff you have to review harder than writing it yourself would have been.

**Small, single-purpose commits with messages that say what changed for the user.** Scroll my git log and you get sentences, not labels:

```
Take the age a worker typed, not a number assembled from its digits
Let only the question on screen answer the question
Read a tax "Yes" back before it costs someone a pension
Stop telling a worker they failed a test we never ran
Stop adding an accident cover to a pension
Refuse a threshold the engine cannot honestly compare
```

Every one of those is a bug fix, and every one names the *consequence to the worker*, not the mechanism. This started as a style preference and became a debugging tool — when the message has to say who it hurts, you notice the ones where you can't answer that.

**Making the assistant break its own fix.** Whenever a test was added for a bug, I'd have the fix reverted deliberately and check the test went red. If it still passed, the test was decorative. This caught more bad tests than anything else I did, and it costs about ninety seconds.

**Using it hardest on the parts I understand least.** I did not write the HTML/CSS for the project site in `livesite/`, and I am not going to claim I did — I specified what it should say and how it should behave, Claude Code wrote the front-end code, and I reviewed and deployed it. Same for the multipart upload internals and parts of the WhatsApp webhook plumbing. The division that actually held: **I own every decision, the assistant owns a lot of the typing.**

### What did not work

**Anything where the assistant had to know a fact about the real world.** Every scheme threshold, age band, rupee figure and exclusion in this project was read out of an official government source document by me, and carries a deep link to the sentence it came from. Not one was authored by an AI. Where a value wasn't researched yet, it was stubbed as `"TODO"` rather than filled with a plausible number — and "plausible number" is exactly what you get if you ask a model for a scheme's premium.

This is not a slight against the tool. It is what the tool is for and what it is not for. A model is very good at "restructure this state machine" and structurally incapable of "what does the PM-SYM page say the contribution is at age 32".

**Review findings that were my own comments read back to me.** When I made the repo public and had an outside model review it, a meaningful fraction of the "findings" were things I had already written down as known tradeoffs in my own code comments, handed back as discoveries. You have to read review output the same way you read anything else — as a claim to check, not a conclusion.

**Advice that contradicted a constraint the reviewer had just praised.** Two separate review rounds told me to add location-based routing so the bot could say "the CSC 4km away" instead of "a CSC" — in the same review where they praised the privacy model. But location routing needs a district or a pincode in the profile, and the privacy model is *specifically* that those fields do not exist.

I did not do it. I wrote down why, and I think being able to say **"I was told to do this and I didn't, here's the reasoning"** is worth more than any claim about who typed which line.

### The honest summary of the AI question

**What the assistants did:** wrote and refactored application code, generated test scaffolding, drafted documentation, built the front-end of the project site, and — via an outside review — found the worst bug in the project, which I had written and not noticed.

**What they did not do:** author a single scheme value, decide the architecture, or make a judgement call.

**What I had to do myself:** decide the four rules above, read the source PDFs, find six bugs by using my own bot like a worker would, notice my impact number was inflated, delete a statistic I had been quoting for weeks, and answer the questions no amount of re-reading a web page will settle — like whether PM-SYM's "₹15,000 or less" includes exactly ₹15,000. That one is a phone call to 14434, not a search.

The through-line: **the value of this system is that it refuses to guess.** That property does not come from the model and it does not come from the tests. It comes from deciding in advance what you are not willing to be wrong about, and then making the code enforce it instead of the documentation.

---

## Part 5 — The bugs

This is the most useful section in the post and the one I'd read first if it were someone else's.

**Not one of these was in the rule engine.** Every single one lived in the layer between the worker and the arithmetic.

### 1. The verification bug — the worst one

Found by an outside review, after I made the repo public.

`Scheme.is_verified` was implemented as:

```python
@property
def is_verified(self) -> bool:
    return not self.stubs
```

So "verified" actually meant "contains no `TODO`".

All three scheme files at the time had been fully researched, so none had a `TODO` left in them — while every one still said:

```toml
verified_by = "unconfirmed — PENDING HUMAN VERIFICATION"
```

Startup printed `PMSBY: verified 2026-08-31`. The engine served real `ELIGIBLE` and `INELIGIBLE` verdicts off values that one person — me — had transcribed from a PDF once, and nobody had checked.

The README said, in bold, not to screen a real worker on this data. **The runtime ignored the README.**

The entire premise of this project is "don't guess". And the program was guessing that *researched* meant *verified*, which are not the same thing at all.

The fix separates them into three properties:

```
is_researched      no "TODO" left — somebody looked it up
is_human_verified  verified_by carries a real signature
is_servable        both — and the ONLY thing evaluate() may gate on
```

**The consequence was immediate and inconvenient: the bot started telling every single worker "I could not check this yet", for every scheme.** And it stayed that way for six days, until I sat down and verified each value at source and signed each file.

> It was a worse demo and a better product.

**The lesson:** I had written that warning in the README, in a comment at the top of every scheme file, and in a test asserting the warning was still present. Three places. All of them prose. The one place it was not written was the code path that decides whether to answer a worker — which is the only place that actually stops anything.

A rule that lives only in documentation is a rule the runtime does not have.

### 2. The double tap that cost someone a pension

Found by me, using my own bot.

Telegram leaves old inline keyboards **live and tappable forever**. My callback payload was a bare `"yes"` carrying no question identity, and `handle()` routed on current state alone. So:

1. Worker taps **Yes** on *"Do you have a bank account?"* → recorded, state advances to `TAX`
2. Nothing appears to happen — slow network — so they tap **Yes** again
3. `handle()` routes on the *current* state, `TAX`
4. They are now recorded as an income tax payer

PM-SYM excludes income tax payers.

**A double tap silently denied a worker a ₹3,000/month pension, and the reply looked completely normal.**

The fix: the adapter now remembers the `message_id` of the keyboard it last delivered, and accepts a callback only from that message, retiring it **before** dispatch. Retiring *before* rather than after matters — multi-select screens like `KNOWN_SCHEMES` and `DOCUMENTS` stay in the same state across taps, so a state-based guard would let a duplicate tap toggle a selection back off.

**The lesson:** this bug was only reachable because of who the user is. On a fast phone with good signal you never double-tap. On a cheap Android over rural 4G, tapping again because nothing happened is not misuse — it is the expected behaviour of the person this is built for.

A correctness bug can be completely invisible until you take the user's context seriously.

### 3. The age parser that repaired instead of rejecting

```python
digits = "".join(ch for ch in answer if ch.isdigit())
```

| Input | Recorded as |
|---|---|
| `9.5` | **95** |
| `-5` | **5** |
| `²` | crash — `isdigit()` accepts it, `int()` refuses |

And nothing in the chat shows the recorded age back, so a worker screened on a number they never typed had no way to see it.

The fix is `answer.strip().isdecimal()` and then a range check — **not** a bare `int()`.

The subtlety is why `isdecimal()` and not something stricter: it rejects `²`, `-5` and `3_4`, while still accepting Devanagari `३४` and Arabic-Indic `٣٤`. A stricter ASCII-only parser would have broken the exact audience this is for.

**The lesson: reject the whole input, never repair it.** Silent repair produces a confident wrong answer, which is the one output this project exists to prevent.

### 4. The six from the first hour of real use

I put the bot live and used it the way a worker would. Six bugs in one session. All six in the conversation layer.

1. **Sending `"reply_markup": null` killed every session.** Telegram answers a null keyboard with a bare `400` and no useful message. The age question and every info command send a message with no buttons — so the session silently stopped dead at the first typed question. Nothing crashed. Nothing logged.
2. **Free-text occupation was a loop with no exit.** "Something else" leads to a free-text box; if the text matched nothing, the code showed the menu again — whose "something else" leads back to the same box. I hit it on the first run by typing "i dont do any job".
3. **No "not working" category, and no Hinglish keywords.** The offline matcher only knew Hindi and English words. Real people type "mistri", "driver ka kaam".
4. **No zero-income band — and PM-SYM's own list would have denied it.** The bands started at "up to ₹5,000". A worker with nothing coming in doesn't recognise themselves in that. Worse, the scheme's band list didn't include the zero case, so a worker with no income was being told they earned too much.
5. **Button labels stayed Hindi in an English session.** The string lookup for income and land bands missed the language argument. The *message* was English and the *buttons under it* were Hindi.
6. **The application pack used `name_hi` and hardcoded Hindi labels.** Same class of bug, one layer further out.

**The lesson:** every test that passed while those six shipped was asserting on **message text**. The bugs were in **button labels**, one layer out.

Test the surface the user touches.

### 5. The methodology bug that only real numbers exposed

Once the scheme files had real values in them, my impact figure jumped in a way that looked too good.

I was summing PMSBY's **₹2,00,000 accident cover** with PM-SYM's **₹36,000/year pension** into one "annual entitlement" figure. That is roughly a **six-fold overstatement**, and a dishonest one — a contingent insurance payout that only pays on a claim and a guaranteed annual pension are not the same kind of money.

They are two separate numbers everywhere now, and the label "annual entitlement surfaced, not money delivered" travels with the figure into the chat message, the printed sheet and the dashboard.

**The lesson:** don't let one number stand in for two different things. And note that the bug only became visible when real values replaced placeholders — **placeholder data hides unit errors.**

This one matters more than a crash. A crash is visible. An inflated impact number is the kind of error that survives all the way to a judge.

### 6. A diagnostic that lied to me

Checking whether the service was really polling, I called `getUpdates` myself, reasoning that Telegram allows only one poller and would return `409 Conflict` if the bot was live. It returned `200 OK`. I concluded the service was down and said so.

Wrong. Telegram **preempts** an existing long poll rather than always returning 409 — so my probe created the very condition it was testing for.

The checks that actually work: `ss -tnp` showing an established connection to a `149.154.166.x:443` address held by the bot's PID, plus fresh `session_start` rows in the event log.

**The lesson: verify with the signal that cannot be faked.** A success code from an API you misunderstand is not evidence.

### 7. Two tests that were green and blind

**Blind #1 — the test that compared the wrong thing.** `test_llm_path_and_button_path_agree` asserted:

```python
assert got == expected, "the LLM changed the result — it must only change phrasing"
```

Both `got` and `expected` were `reply[0].text`. But `reply[0]` is the **answer recap**, which is built from the profile and is identical whatever the engine decided. `reply[1]` is the verdict. The test never looked at the result it claimed to protect — I could replace the verdict with a sentinel string and it still passed.

**Blind #2 — the walk that stopped walking.** Adding the stale-keyboard guard broke `test_all_paths.py` invisibly. The test fabricated `message_id: i + 1` instead of using the id the wire actually returned, so under the new guard **all 12 taps were rejected and no session was ever created.** Every command was being tested against the opening screen.

The suite still printed `all checks passed`.

**The lesson: a green suite after a behaviour change is a claim, not evidence.** When a fix invalidates a test's assumptions, repairing the test is part of the fix, not a footnote.

### 8. Three bugs that only appeared when I ran it as a real service

I wrote `deploy/sathi.service` days before running it. Installing it for real found three things in ten minutes:

1. **No `PYTHONUNBUFFERED=1` in the unit.** Python block-buffers stdout when it's a pipe, and journald is a pipe. `journalctl -u sathi -f` showed *absolutely nothing* for a bot running perfectly. I spent real time convinced the service was dead. The Dockerfile set this; the unit never did.
2. **`StartLimitIntervalSec` and `StartLimitBurst` were in `[Service]`.** They are `[Unit]` directives. systemd logged `Unknown key … ignoring`, and the crash-loop rate limit my own comment promised simply did not exist.
3. **`run-locally.sh` did not restart anything.** It ran `systemctl enable --now`, which only *starts* a stopped service. Re-running it on a live bot left the old process running the old config — so an edit looked applied and wasn't.

None of those were hard. All were invisible until the thing actually ran.

### 9. The retry loop that was a hot loop

`run_forever()` caught network errors and retried immediately with no delay. For a dropped connection that was fine, because `getUpdates` blocks for 50 seconds and paced the loop for free. For a **permanent** error it was not — a revoked token returns `401` instantly, so the loop became a spin that would peg a CPU core and write the same log line forever.

Now: 1s → 60s exponential backoff, reset on the first success.

### 10. The same bug, one layer up

This is my favourite of the lot, in the way a bug becomes your favourite when it teaches you something about yourself rather than about the code.

The Uttarakhand old-age pension and the widow pension are alternative routes to the *same* state payment. A 65-year-old widow qualifies for both; the state pays one. I had already met this problem and solved it. Scheme files carry `exclusive_group = "uk_state_pension"`, and `engine.value_totals()` collapses a group to its largest member before summing. The worker sees ₹18,000. Correct.

The **dashboard** did this:

```sql
SELECT scheme_code, SUM(value_inr)
FROM events
WHERE event_type='scheme_newly_surfaced'
GROUP BY scheme_code
```

No sessions. No groups. That same widow contributes ₹18,000 twice, and the impact page reports **₹36,000 of annual entitlement surfaced** for one woman entitled to ₹18,000.

Read the direction of that failure carefully. The worker was told the truth. The number *about* the worker was inflated. That is the worse way round, because an inflated statistic is never caught by using the product — it is caught by someone reading the code, or by nobody at all, and it ends up in a submission form.

I wrote `exclusive_group` specifically to prevent this. Then I wrote a second place that computes the same quantity and did not use it. **A fix that lives in one function protects one caller.** `templates.py` and `pack.py` were both made to call `value_totals()`; `report.py` never was, because it reads the event log rather than a live result, and I never made the connection.

The fix groups per session. The second half of the test matters as much as the first:

```python
assert split["payout"] == 54000   # one widow: two pensions collapse, plus PM-SYM
assert split["payout"] == 72000   # two DIFFERENT widows: still two entitlements
```

Collapsing is not deduplication. Sessions are unlinkable by design, so two visits by one person still count twice — a real limitation, but a separate and disclosed one.

Found by an outside review reading the repository. Not by 2,541 walked paths.

### 11. I built a privacy feature and leaked it at the proxy

The application sheet used to be sent as an HTML file. WhatsApp refuses `text/html`, so that channel got a flattened text version and lost the layout entirely. The fix was to serve the sheet as a link.

I was careful about it. Tokens are `secrets.token_urlsafe(16)`. They live in a dict in memory and never touch disk, because `pack.py` promises exactly that. They expire in an hour. `/clear` revokes them. Responses carry `Cache-Control: no-store`, `Referrer-Policy: no-referrer`, `X-Robots-Tag: noindex`. And the Python server suppresses its own access log, with a comment saying why:

> The default access log records the path, which is the token, and the client address. Logging which pack was opened from where is exactly the per-person analytics this project promises not to keep.

Then I put Caddy in front of it for TLS, using the config from my own runbook:

```
yojanasathi.avinashnegi.com {
	log
	handle /p/* { reverse_proxy 127.0.0.1:8081 }
}
```

Caddy sees the request **first**. Within minutes its log held lines shaped like:

```
"remote_ip":"203.x.x.x"  "uri":"/p/mBhrQ5…"
```

A client IP, next to a currently valid bearer link to a stranger's benefits sheet. The inner server's refusal to log it was worth nothing at all.

Only my own dead test tokens were ever in there, because no worker has used the feature yet. That is luck, not design.

The lesson is not "remember to configure the proxy". It is that **a privacy property enforced inside one process is not a property of the system.** Every hop that can see the data has to agree, and the hop I forgot was one I had added myself, that same afternoon, and written the runbook for.

So it is now enforced at the proxy rather than left to me to remember:

```
format filter {
	fields {
		request>uri regexp "/p/[^\s?#]+" "/p/REDACTED"
		request>remote_ip delete
	}
}
```

### 12. A file that argued against its own signature

`data/schemes/uk_widow.toml` is the weakest-sourced of the seven. The original rate citation now 404s. The department's own widow page states no amount. myScheme — the one page that does give ₹1,500/month — links to a *different scheme* as its "Official Website". I read all of it three times across two days and wrote every finding into the file.

Including this line:

> `# ! This is why this file stays unsigned while the other six are signed.`

And then I signed it. `verified_by = "Avinash Negi, checked 2026-09-10"`, four lines below.

Both statements were true when written. The comment was the state on the 9th; the signature was the decision on the 10th. Nothing forced them to agree, so they didn't, and the file spent a day telling every reader that its own signature should not exist.

An external reviewer found it and called it a trust-model contradiction, which is exactly what it is. In most projects that is a documentation bug. Here, provenance **is** the product — the entire argument is "these rules are not AI guesses, here is the source" — so a file contradicting its own provenance attacks the central claim.

I kept the three findings. Evidence that a source was checked three times is the most valuable thing in that file. What I replaced was the conclusion, which now records the decision instead of denying it: signed knowingly, the rate rests on a single non-primary government source, the "not receiving another pension" bar was **removed rather than encoded** because no department page establishes it, and withdrawing the figure is one edit — set `annual_value_inr = "TODO"` and the scheme returns UNKNOWN.

Its sibling `uk_old_age.toml` had the same disease plus two statements that had simply become false: it claimed a condition was "now encoded below" that had since been removed, and claimed the income question asked the stricter of two readings when it asks the other one.

**Comments describing a decision rot the moment the decision changes**, and no test suite checks prose. The only defence I have found is to write down *why* rather than *what*, and to re-read the header every time the body changes.

### 13. `/clearall` took three minutes to answer

I sent `/clearall` at 6:37 and got the reply at 6:39.

Telegram's `deleteMessages` accepts up to 100 ids at once. When a batch was refused — almost always because those messages are older than the 48-hour deletion window — my fallback retried **one API call per id**. Four hundred ids meant up to a thousand requests, on the polling thread, so the worker's next answer queued behind all of it. Telegram then flood-limited the bot, which is why the conversation stayed slow long after the command had finished.

The fix is a budget on *futile* work rather than on work:

```python
_WASTED_DELETE_BUDGET = 25     # consecutive deletes that found nothing
_EMPTY_CHUNKS_BEFORE_STOP = 2  # consecutive fruitless chunks before giving up
```

Counting futile calls rather than total calls matters. An older Bot API with no `deleteMessages` still clears the entire window, because those single calls succeed — only the pointless ones spend budget.

The docstring had always claimed the walk "stops once deletes stop working". The batched path never did. **The comment described the behaviour of code that had since been rewritten** — the same failure as #12, in a different file, found in the same week.

### 14. A custom domain silently switched off the live counter

The landing page reads a small `/stats.json` from the running bot and shows what has actually happened. It is written to hide itself if the fetch fails, so the page degrades to precisely what it was before the feature existed.

Then I pointed a custom domain at GitHub Pages. GitHub applies a user-level custom domain to **every** project page, so the site moved from `avinashnegi1999.github.io/yojana-sathi/` to `avinashnegi.com/yojana-sathi/` and the old URL began 301-ing.

The CORS allowance still named only `github.io`. The browser blocked the request. The section hid itself.

**The failure mode worked perfectly, for entirely the wrong reason.** That is the part worth keeping. A graceful degradation is also a silent failure. I caught it only because I happened to reload from the new address a minute after DNS propagated; a skeleton or a spinner would have screamed. Hiding was the right call for a worker and the wrong one for me.

One header cannot name two origins, so the server now echoes back whichever allowed origin asked, with `Vary: Origin` so a cache cannot replay one origin's response to another. Never `*` — the endpoint answers strangers.

### The pattern across all of them

| | |
|---|---|
| Where the bugs were | conversation layer, adapter, dashboard, reverse proxy, comments, tests, deployment |
| Where the bugs were **not** | the rule engine |
| Most common shape | something looked verified/tested/correct without being it |
| Most common cause | a check that measured an adjacent thing |

The engine survived because it is small, pure, and has no I/O. Everything that touches the outside world is where the defects live — which is a decent argument for keeping the decision-making core as small as you can get away with.

---

## Part 6 — Things the research forced me to change

I wrote the scheme file format first and then went and researched the actual schemes. Three assumptions in my format did not survive contact with the sources.

**e-Shram is not a benefit.** I had assumed every scheme has a ₹ value. e-Shram is a *registration* that issues a UAN card; the benefits are delivered *through* it. Giving it a rupee figure would have double-counted the PMSBY cover it unlocks. I added `value_basis = "gateway"` so it can be surfaced as valuable without contributing a number.

Recording that e-Shram is worth ₹0 of its own is not a gap in the data. It is the correct behaviour, and it is a lot better than inventing a figure.

**One yes/no question covered three different memberships, and that was wrong.** PM-SYM excludes members of EPFO, ESIC and NPS. e-Shram's own definition of an unorganised worker excludes EPFO and ESIC but says nothing about NPS. I used a single `is_statutory_scheme_member` field for both, which made the system stricter than e-Shram's own wording for a worker holding NPS alone.

I had written that down as a deliberate tradeoff and told myself that under-promising was the safe direction. **It is not.** e-Shram is the gateway the other schemes are delivered through, so a wrong NO there is a missed entitlement — and "stricter than the source says" is a correctness failure whichever direction it points.

Split into `is_epfo_or_esic_member` and `is_nps_member`. Two fields, not three, because no scheme distinguishes EPFO from ESIC and every extra field is another question a worker has to answer on a phone.

**The part that makes this worth telling:** my own boundary test could not have found it. I have a test that re-encodes each scheme's rules from the official source text, independently of the data files, and compares that oracle against the engine across every combination. But the oracle had copied the same `statutory = EPFO or ESIC or NPS` abstraction from the production model — so both sides of the comparison were wrong in the same way, and ten thousand verdicts agreed perfectly.

> An oracle is not independent because it lives in another file. It is independent when it is **free to disagree** — which means expressing the rules the way each *source* words them, not the way the code stores them.

The oracle now names `epfo_or_esic` and `nps` separately per scheme, and `test_nps_alone_disqualifies_pm_sym_but_not_eshram` fails if the two are ever merged again. I checked that by reintroducing the bug on purpose and watching the test go red.

**A premium is not always a number.** PM-SYM's monthly contribution depends on the age you join at — ₹55/month at 18, ₹200/month at 40. The field now accepts a short sentence as well as an integer. It's never summed, only shown, so prose is safe there. But it must be non-empty, because an empty string reads as "free".

---

## Part 7 — The numbers in my own README that were wrong

The problem statement is worth 50% of the score, and it was the only part of the repo with no sources. When I went to add citations, **three of my four headline numbers did not survive:**

- **44 crore → 43.99 crore.** Close, but the real figure has a source: Economic Survey 2021-22 via a PIB Lok Sabha reply.
- **93% → 82.7%.** The number I had was simply wrong. The last full survey found 39.14 crore of 47.41 crore, which is 82.7%.
- **₹6,688/month — deleted entirely.** No source exists. It entered my planning document uncited and was never real. **I had been quoting it as fact for weeks.**
- **31.38 crore → 31.48 crore**, as on 26 January 2026, with a PIB release id.

Deleting ₹6,688 turned out to be an upgrade, not a loss. The replacement is the **daily** wage figure from the 2025 labour force survey — ₹455 male, ₹315 female — which actually backs the core claim of the whole project.

"A wasted trip to a CSC costs a day's wages" stops being rhetoric the moment you can name the wage.

**The lesson:** cite the problem statement before you write the solution. I built the whole system on a number that does not exist. It changed nothing about the code — but if a judge or an interviewer had asked where it came from, I'd have had no answer.

I now keep an explicit **"numbers not to quote"** list in the companion handbook, of figures that have appeared in AI-generated summaries of this project and are wrong. Things like "44M+ unorganized workers" (unsourced), "zero-persistence architecture" (false — the database has two tables), and "guarantees 0% hallucination" (the model is optional and off; that's a design choice, not a proof).

Quoting an unverifiable number on a résumé is the exact failure this project exists to prevent. An interviewer who opens `schema.sql` and finds two tables has caught a lie about the one thing the project claims to care about.

---

## Part 8 — Shipping it

### Deployment was parked for days behind an estimate that was wrong by an order of magnitude

The bot ran on my laptop for three days. Every plan to fix that went through Azure for Students, which was stuck behind academic verification my university email couldn't satisfy, and before that Fly.io, which was stuck behind a card preauth that Indian cards routinely decline for international transactions.

Two vendors, one wall. **The wall was never the vendor. It was the card.**

Asking "can I use AWS instead" turned out to be the right question for the wrong reason: AWS needs the same card, but I already had an account carrying $120 of credit. Nothing about the card was solved. It simply stopped mattering.

`deploy/provision-aws.sh` mirrors the Azure script rather than replacing it: imported SSH key so the provider never holds the private half, one address allowed on port 22, the current Ubuntu image resolved from Canonical's SSM parameter instead of a hardcoded AMI id that rots, encrypted volume, IMDSv2 required, and a zero-spend budget that stays silent while credit covers the bill and mails the day it stops. It refuses to build a second instance if one exists, because two pollers on one bot token steal each other's updates.

`install-on-vm.sh` needed **one line changed** — the key path. Everything else was already host-agnostic, which is the payoff for having written the systemd unit to be identical on a laptop and a VM.

Two bugs, both found by running it rather than reading it. AWS rejects non-ASCII in a security group description, so an em dash killed the call. And the budget alert email had been hardcoded into a file destined for a public repository.

**The whole thing took under an hour**, most of it waiting for an instance to boot.

The estimate that had kept it parked — that shipping needs a clear day — was wrong by an order of magnitude, and it had been wrong the entire time it was preventing the work. I had already learned this once on Saans and written it down. Writing it down was not enough.

> **Deploy before you polish.** Building feels like progress and shipping feels like a task that needs a clear day. Only one of them is measured.

### What live actually looks like

| | |
|---|---|
| Host | AWS EC2, `ap-south-1` |
| Path | `/opt/sathi` — a plain file copy, **not** a git checkout |
| Service | systemd unit `sathi`, user `sathi` |
| Env | `/etc/sathi/sathi.env` |
| Cost | ~$10.50/month against $120 of credit |
| Logs | `sudo journalctl -u sathi -f` |

Deploying is `rsync`, run the suite **on the target**, back up, swap, restart:

```bash
rsync -az --exclude '.env' --exclude '*.db' --exclude '__pycache__' \
  sathi data tests check.py pyproject.toml \
  ubuntu@<host>:/tmp/sathi-new/

ssh <host> 'cd /tmp/sathi-new && python3 check.py'
ssh <host> 'sudo cp -a /opt/sathi /opt/sathi.bak-$(date +%F-%H%M)
            sudo rsync -a --delete /tmp/sathi-new/ /opt/sathi/
            sudo chown -R sathi:sathi /opt/sathi
            sudo systemctl restart sathi'
```

Running the suite on the target matters: the VM runs Python 3.12 and my laptop runs 3.11, and that difference is exactly where a deploy-only failure hides.

Every deploy makes a timestamped backup first, so rollback is one command. That is the only reason deploying at 11pm is acceptable.

And I rebooted the instance on purpose to watch the bot come back by itself. It did.

**What deploying did not do** is worth saying plainly, because a running service is very easy to mistake for a finished one. At that point no scheme had a human signature, so the engine answered `UNKNOWN` to every worker by design.

A bot that is up continuously and helps nobody is still up continuously.

### WhatsApp

Telegram was first because long polling needs one outbound connection and nothing else. A webhook needs a public URL, TLS termination, and a platform that stays awake — three things that can fail on deploy day.

WhatsApp came later and speaks the **same conversation from the same rule engine**, through `channels/whatsapp.py` and a shared `channels/router.py`. It runs behind Caddy for TLS on the same instance, with a permanent System User token so it doesn't expire out from under me, and a rotation script that moves the token without it touching a disk.

It is built and verified end to end against Meta's test number — a real phone, a full screening, the same Hindi — but it is **not yet live on a public number**, because a production WABA needs Meta Business Verification and the test number is capped at five recipients.

There's a small tool I'm fond of that came out of this:

```bash
python3 -m sathi.main --preview whatsapp
```

It renders exactly what the wire would carry, in the terminal, with no token and nothing sent. Being able to see the keyboard without reaching for a phone made iterating on the flow about five times faster.

**Why the channel was separable at all** is not architectural purity. It's schedule risk. Meta verification can take weeks and can fail. Making the channel swappable meant a verification delay could never block a deploy date. De-risking a calendar is a better justification than "it's cleaner".

### The rest of the infrastructure

- **GitHub Actions** runs the full suite and the image build on every push (`.github/workflows/checks.yml`).
- **Docker** as an alternative deploy path, running the whole suite at image build time. `DB_PATH` must point at a mounted volume — a free-tier container that loses its disk on restart loses the event log and every impact number with it.
- **A project site** in `livesite/`, published by GitHub Actions to a custom domain: walkthrough video, real screenshots of both channels, the architecture, live numbers from the running bot, and a section on what this does *not* claim.

---

## Part 9 — Keeping the rules honest after they're written

Two tools, both standard library, neither of which the bot itself runs. These are the parts I'd point at if someone asked what makes this more than a chatbot.

### `python3 -m sathi.review PMJJBY` — signing a scheme off

This is the **only supported way a name reaches `verified_by`.**

It prints every value the engine will use, beside the URL it came from, one scheme per screen. You open that page. If it all matches, you type the scheme code and your name.

It then writes **exactly two lines** — and a self-check signs a real copy and asserts that precisely two lines differ, so a signature can never carry a changed threshold in with it. It refuses to run without a terminal, and it refuses a name that looks automated. `--unsign` reverses it.

Nothing in that tool checks anything for you. It puts the values and the source on one screen so *you* can.

### `python3 -m sathi.sources` — has a ministry changed a number?

This answers the question a signature cannot: a person checked this in September, but is it still true?

`data/sources/` holds a fingerprint of each official page plus **43 named claims** — one per value we rely on — and re-reads the live pages on demand. It also watches for things that must *not* reappear, like the "16–59" age limit e-Shram no longer states.

`data/sources/official-text/` keeps the pages themselves, so a rule can be audited without leaving the repository. That turned out to matter more than I expected: **one URL I cited in the morning had 404'd by the afternoon**, two of these hosts refuse an ordinary fetcher, and one official source is a 320 MB scan.

### The sign-off week

Between 9 and 10 September I sat down with the official pages and signed all seven scheme files: PMJJBY, PMSBY, PMUY, the Uttarakhand old-age pension, e-Shram, PM-SYM, and the Uttarakhand widow pension. Each one is a separate commit, and each commit message names what the reading actually changed:

```
Sign PMJJBY, and stop the tests assuming nothing ever is
Sign PMSBY, and say what it does not pay for
Sign PMUY, and stop offering it to houses with piped gas
Follow the department, not the aggregator, on its own pension
Sign e-Shram, and tell farm workers it is for them
Sign PM-SYM, and say that no income proof is needed
Record why the widow pension is the one that stays unsigned
Sign the seventh, and let no aggregator refuse anyone
```

**One file is thinner than the others, and the repository says so.** The Uttarakhand widow pension's own department page states no amount at all. The ₹1,500 figure comes from myScheme — whose "Official Website" link for that scheme points at a *different* scheme. It is signed on my judgement, and `data/schemes/uk_widow.toml` records exactly that in its header. Two questions to the SSP helpline would settle it.

And where a source refuses people without saying so anywhere official, this project declines to. Both Uttarakhand pensions ask "are you already drawing another pension?" and then **tell the worker to ask at the office** rather than refusing her on it — because only myScheme states that bar, and a wrong NO is a pension nobody claims.

That signature is enforced, not merely recorded. Until a named human signs a file, the rule engine returns `UNKNOWN` for that scheme to every worker, contributes ₹0 to every number on the dashboard, and says so at startup. And `tests/test_schemes.py` **fails the build if the README and the data ever disagree about which schemes are signed.**

---

## Part 10 — Testing

| | |
|---|---|
| Command | `python3 check.py` |
| Module self-checks | 20, run in dependency order |
| Test files | 15 |
| Framework | none — plain `assert`, nothing to install |
| Runtime | a couple of minutes |

Self-checks run **in dependency order**, so the first failure points at the lowest broken layer instead of the five things downstream of it.

There are two tests doing most of the work, and they find completely different things.

**`tests/test_all_paths.py`** presses every button at every reachable screen in both languages. As of the run I did while writing this it reported **2,541 paths and 217 completed sessions per language**, and across the whole walk: 6,306 replies, 26,322 buttons, 432 generated packs, 141,624 event rows, and 2,414 + 23,908 WhatsApp buttons and list rows. It opens every generated sheet, runs every path through a real event log, drives every command through the channel adapter, and fuzzes the typed questions.

**`tests/test_rule_boundaries.py`** asks whether the **answers** are right, not whether the code runs. Each scheme's rules are re-encoded from the official source text, independently of `data/schemes/`, and compared against the engine across every combination of the fields any rule touches — **214,326 verdicts**, plus every named threshold one per line.

> Pressing every button can never find a wrong threshold, because a wrong threshold renders a perfectly well-formed screen.

That sentence is why both tests exist.

### The defence: tests that fail when they stop testing

After the six-bug session, the walk got **coverage counters it asserts on itself**:

```python
assert CHECKED["packs"] >= 2, f"no pack was ever generated or checked: {CHECKED}"
```

That assertion paid for itself almost immediately. Enforcing the verification gate meant no scheme was servable, so no worker reached an eligible result, so no pack was ever generated — and the walk quietly fell to a fifth of its paths while still passing every assertion about the screens it *did* reach.

The counter caught it.

**A test that silently stops reaching the thing it checks is worse than no test, because it produces confidence instead of doubt. Make it assert its own reach.**

### Privacy, tested rather than promised

`tests/test_privacy.py` drives every event type through the log and then asserts, column by column, that nothing else survived. That test is the reason the privacy claim is defensible rather than aspirational.

What is actually stored: `sathi.db`, SQLite, two tables — `events` and `followups`.

I do **not** say "zero persistence". It's false and `schema.sql` is one click away. The accurate claim is stronger:

| Property | Detail |
|---|---|
| Granularity | coarse bands only, never raw answers |
| Identity | random per-session id, **not** derived from the messaging account id |
| Aggregation | k-anonymity suppression below n=5 |
| Writer | `metrics/events.py` — the only module that may write |

A single writer matters: the privacy invariants are enforced in one file you can read in full, not scattered across every call site.

Consent runs **before anything is recorded**, and declining is a real path that stores nothing and ends the conversation. There's a test for that too.

And there is **no auto-submission**. The bot generates an application pack; the human submits it. There is no code path that files anything with a government portal on a worker's behalf. An agent that can act irreversibly for someone who cannot read the confirmation screen is a different and much more dangerous product.

---

## Part 11 — Where it honestly stands

I keep a scored breakdown in the repo, and the point of writing it down is the low score, not the high ones. A scorecard that flatters everything says nothing.

| Dimension | Score |
|---|---|
| Idea / problem | 9 / 10 |
| Architecture | 9 / 10 |
| Testing | 9.5 / 10 |
| Safety / correctness | 9.5 / 10 |
| Documentation | 9 / 10 |
| Code / engineering | 9 / 10 |
| **Current product maturity** | **7 / 10** |
| Portfolio value | 9 / 10 |

**The 7 is the honest number and the reason the rest is worth reading.** The engineering is ahead of the product.

An outside review a week later put the engineering at 8.9/10 and then estimated the same project at **65-68/100** against the hackathon's own rubric — because proven impact is 25% of that rubric and mine is worth about 3 of those 25. Both numbers are fair, and the gap between them is the whole story. It is not "build more". Nothing in the second number is a code problem.

What's missing is not code:

- **No real worker has completed a screening.** Five screenings have run on the live bot. All five are mine. Nobody outside the build has used this end to end, and no amount of engineering closes that gap — it needs a person to walk into a room with a phone.

  The landing page reads that count from the bot and prints it, captioned *"still maintainer testing, not a field pilot"*. It renders nothing at all until the count is above zero, because a live counter reading `0` claims less than the honest sentence sitting underneath it.
- **The Hindi and English strings have never been read by a native speaker.** I can build the pipeline; I cannot certify the register of a language I'm writing *for* someone else to hear.
- **Seven schemes.** National coverage is vastly larger. The authoring path is documented, so adding more is research effort, not engineering effort.
- **WhatsApp is built but not on a public number.** Meta Business Verification needs a phone number that has never had WhatsApp installed on it, which costs a second SIM and about ₹1,800 a year to keep alive. I decided against buying one for a channel I am not launching this month. The code stays built, tested and documented as not-public — which is a deployment boundary rather than a gap, and I would rather write that sentence than imply otherwise.

  It is the channel my actual users are on, though, and I know it. Every worker recruited to Telegram has to install an app first. If a partner ever tells me their workers will not do that, the ₹1,800 stops being a cost and starts being obvious.
- **The follow-up sender is designed and unbuilt.** Storage and purge exist; the sender does not.

There's one open research question I want to name because it is the exact shape of thing this project is built to handle: the PMSBY age cap reached *through* the e-Shram route. e-Shram itself has no upper age bound in the visible FAQ, but a `59` appears inside an unclosed HTML comment in Question 40 and caps PMSBY via that route. **That needs a phone call to a CSC operator, not another reading of the FAQ.**

And one known technical limitation, documented in the code and deliberately not patched over: after a transient Telegram failure, the worker's buttons go dead. Restoring the keyboard would let the old question's buttons answer the new state — the exact bug the message-id guard exists to prevent. A real fix means advancing the `getUpdates` offset only after a turn commits, which risks double-writing events.

What *did* land since I first wrote this section, all of it deployment rather than features: the bot now serves the application sheet as a link on its own subdomain with a real certificate, so a worker opens it in her phone's browser and the print menu offers Save as PDF — the sheet no longer arrives as a file Android often cannot render, and WhatsApp will not have to receive it as flattened text when its turn comes. Tokens expire in an hour, die on restart, and `/clear` revokes them. There is a landing page with a walkthrough video and live counters. None of that moves the number that matters.

**Every item on that list is a person-shaped blocker** — a language review, a recruited user, a phone call, a verification queue. None of them is "the code doesn't work".

For a portfolio project, that's a good place to be stuck, and I'd rather say it plainly than apologise for it.

There is also a non-technical risk sitting above all of this that I have no control over: the hackathon's submission form has a required checkbox saying the project was built using AgentFoundry, the official IDE. This was built locally in Python. I have written to the organisers asking whether importing an existing repo and continuing there qualifies, and I have no reply yet. I've documented the whole question in `docs/AGENTFOUNDRY_MIGRATION.md` rather than quietly ticking a box, which is the only version of this I'd be comfortable defending.

If the answer turns out to be no, the project loses a hackathon and keeps everything else: a live bot, seven scheme files with a citation on every value, and a rule engine I can explain to anyone. I would rather have that and no entry than an entry resting on a checkbox I ticked without believing it.

---

## Part 12 — What I'd tell someone starting

**Decide what the expensive mistake is, then build around it.** Not "what features should this have". One sentence: what is the wrong output that costs the user the most, and in what unit? Everything downstream gets easier once that's written down.

**Make the unfinished state un-representable.** `"TODO"` in a numeric field. A missing column instead of a privacy promise. An import that isn't there instead of a code review rule. Every time I moved a safety property from "remember this" to "the type system stops you", it stayed fixed. Every time I wrote it in prose, it rotted.

**Ship it before you polish it.** I learned this on Saans, wrote it down, and then repeated the mistake on Scheme Sathi anyway. Deployment took under an hour. It had been parked for days behind an estimate that shipping needs a clear day. Building feels like progress. Only shipping is measured.

**Use your own thing like the person you built it for.** Six bugs in the first hour. The engine — the part I'd tested hardest — was fine. Every bug was in the buttons.

**Test the surface the user touches, not the surface that's convenient to assert.** My tests all checked message text. The bugs were all in button labels. And make each test assert its own reach, so it fails when it stops testing anything.

**Cite the problem before you build the solution.** I built the whole thing on a ₹6,688/month figure that does not exist anywhere.

**On using an AI assistant:** let it do the typing, not the deciding, and be exact about the line. Every architectural call, every judgement call, and every number in this project is mine and defensible. The front-end code is not mine and I say so. Both of those statements make the project easier to talk about, not harder — and the second one is why anyone should believe the first.

The through-line of everything above is the same: **the value of this system is that it refuses to guess.** That doesn't come from the model, and it doesn't come from the tests. It comes from deciding in advance what you are not willing to be wrong about, and then making the code enforce it instead of the documentation.

---

## Links

- **Code:** `github.com/avinashnegi1999/yojana-sathi` — Apache-2.0
- **Try it:** [@YojanaSathiBot](https://t.me/YojanaSathiBot) on Telegram
- **The site:** avinashnegi.com/yojana-sathi
- **The handbook:** `github.com/avinashnegi1999/scheme-sathi-handbook` — 19 chapters on the same material, written for me to revise from
- **The unedited version:** `docs/BUILD_LOG.md` in the repo — every bug, every wrong turn, nothing tidied up afterwards

---

### Stack

Python 3.11+ (standard library only, zero third-party dependencies) · TOML for scheme rules · SQLite for the event log · Telegram Bot API and WhatsApp Cloud API, both written directly against `urllib` · systemd on AWS EC2 · Caddy for TLS · Docker as an alternative path · GitHub Actions for CI and Pages · plain `assert` for tests, no framework
