# Drill

Ten questions someone can ask after ten minutes in the repo, with the answers.

**Cover the answer, say yours out loud, then check.** Reading these and nodding
proves nothing — the failure mode is knowing something and not being able to say
it under pressure.

---

### 1. Walk me through what happens when a user sends `/start`.

`handle_update` receives the update and decides message vs callback. It hands
the answer to `_dispatch`, which finds or creates a `Conversation` for that chat.
`Conversation.handle` looks at `self.state` and calls the matching `_on_<state>`
method by name — `getattr(self, f"_on_{self.state.value}")`. That method records
the answer, sets the next state, and returns a list of `Reply` objects. Back in
the adapter, `send` turns each `Reply` into an API call.

> *Follow-up:* "What if there's no matching method?" — `handle` returns a
> generic error reply and ends the session rather than raising.

---

### 2. Open `evaluate()`. What is the precedence, and why that order?

1. Not servable → `UNKNOWN`, always, no further checking
2. Any exclusion definitely hits → `INELIGIBLE`
3. Any criterion definitely fails → `INELIGIBLE`
4. Anything undecidable → `UNKNOWN`
5. Otherwise → `ELIGIBLE`

A definite NO outranks a maybe, so 2 and 3 sit above 4. Rule 1 is above
everything because a scheme nobody has finished must not produce a verdict at
all — not even a negative one.

---

### 3. Why is `UNKNOWN` not just a `NO`?

The errors are not symmetric. A wrong NO costs a worker a scheme they were
entitled to, possibly for years. A wrong YES costs a day's wages and the fare.
`UNKNOWN` carries a question to ask at the centre, so they still leave with
something usable. Turning someone away on unchecked data is the expensive
direction.

---

### 4. What is `is_servable`, and why does only `evaluate()` use it?

`is_researched` (no `TODO` left) and `is_human_verified` (`verified_by` carries
a real signature); `is_servable` is both.

**This is the bug story.** `is_verified` used to be `return not self.stubs`, so
"verified" meant "contains no TODO". Every file was researched, so the gate
passed — while all of them still said `PENDING HUMAN VERIFICATION`. The bot
served real verdicts off values one person transcribed from a PDF once.

---

### 5. The government changes a benefit amount. What do you edit?

`data/schemes/pm_sym.toml` — change the field, update `source_url` to the deep
link proving the new number, set `verified_on` to today, re-sign `verified_by`.
No Python touched; the engine reads the file. That is why rules are TOML: the
person who knows welfare policy should not need to know Python.

---

### 6. Why can't `sathi/rules/` import an LLM?

Compute, then narrate. The model runs conversation flow, maps free text to a
category — always confirmed back before it is recorded — and rephrases Hindi a
human wrote. It never sees a threshold, never produces a rupee figure, never
produces a verdict. A model right 95% of the time is wrong once in twenty, for
someone spending a day's wages.

> *Strong follow-up:* `LLM_API_KEY` unset is a **tested** configuration. Every
> eligibility result is identical with or without it.

---

### 7. How do you know it doesn't break?

`python3 check.py` — 15 module self-checks, 6 test files, no framework.
`test_all_paths.py` walks 486 paths per language and asserts its own coverage
counters, so a test that stops reaching what it checks fails rather than passing
quietly.

> **Volunteer the limit:** this suite has twice been green while blind. Once an
> acceptance test compared answer recaps instead of verdicts and passed with a
> corrupted result. Once a walk fabricated message ids and tested every command
> against the opening screen. Both fixed. A green suite is a claim, not evidence.

Saying this before they find it is what separates understanding a system from
reciting it.

---

### 8. What data do you store?

An event log — `sathi.db`, tables `events` and `followups` — coarse bands only,
under a random per-session id **not** derived from the Telegram id, with
k-anonymity suppression below n=5. `metrics/events.py` is the only writer.

> **Never say "zero persistence".** It is false and `schema.sql` is one click
> away. The correct claim is stronger: `Profile` has no name, phone or Aadhaar
> field at all.

---

### 9. Tell me about a hard bug.

Lead with the verification bug (Q4). If they want a second: tapping Yes twice on
the bank question recorded the worker as an income tax payer, which excludes
them from PM-SYM — a double tap silently denied someone a ₹3,000/month pension
and the reply looked normal. Telegram leaves old keyboards live and the callback
carried no question identity. Fix: accept a callback only from the keyboard last
delivered, retired before dispatch.

---

### 10. Did you use AI to build this?

Yes, documented openly in `LESSONS.md` and `BUILD_LOG.md`. The claim is not that
every line was typed by hand; it is that the judgement calls are defensible —
model out of the engine, `"TODO"` over `0`, no unsigned value served, PII
structurally impossible. Plus reviews where the advice was wrong and wasn't
taken.

Then offer: *"give me a scheme change and I'll show you where it goes."*

---

## If you do one thing

Put a `print()` at the top of `handle_update`, `_dispatch`, `Conversation.handle`,
`_on_age`, and `evaluate`. Run one screening. Watch the order.

Ten minutes, and Q1 stops being something you recite.

---

Next: [15-numbers.md](15-numbers.md)
