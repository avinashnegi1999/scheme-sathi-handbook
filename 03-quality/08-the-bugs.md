# The bugs

The most useful chapter in this handbook. Every one of these is a real defect
that shipped or nearly shipped, with what it cost and what it taught.

**Not one of them was in the rule engine.** They were all in the layer between
the worker and the arithmetic.

---

## 1. The verification bug — the worst one

**Found by:** an outside review, after making the repo public.

`Scheme.is_verified` was:

```python
@property
def is_verified(self) -> bool:
    return not self.stubs
```

So "verified" meant "contains no `TODO`". All three scheme files had been fully
researched, so none had a `TODO` left — while every one still read:

```toml
verified_by = "unconfirmed — PENDING HUMAN VERIFICATION"
```

Startup printed `PMSBY: verified 2026-08-31`. The engine served real `ELIGIBLE`
and `INELIGIBLE` verdicts off values one person had transcribed from a PDF once
and nobody had checked. The README said in bold not to screen a real worker on
this data. **The runtime ignored the README.**

### The fix

```
is_researched      no "TODO" left — somebody looked it up
is_human_verified  verified_by carries a real signature
is_servable        both — and the ONLY thing evaluate() may gate on
```

### The consequence — say this part

**The bot now tells every worker "I could not check this yet" for all three
schemes,** and stays that way until each value is verified at source and signed.

> It is a worse demo and a better product.

### The lesson

Writing a safety rule in prose is not the same as enforcing it. The README was
correct and completely inert. A rule that lives only in documentation is a rule
the runtime does not have.

---

## 2. The double tap

**Found by:** the author, using his own bot.

Telegram leaves old inline keyboards live. The callback payload was a bare
`"yes"` with no question identity, and `handle()` routed on current state alone.

Tap **Yes** on the bank question, tap it again because nothing seemed to happen
— the second tap answers the *tax* question. PM-SYM excludes income tax payers.

**A double tap silently denied a worker a ₹3,000/month pension, with a reply
that looked entirely normal.**

### The lesson

The bug was only reachable because of who the user is. On a fast phone with good
signal you never double-tap. On a cheap Android over rural 4G it is the default
behaviour. **A correctness bug can be invisible until you take the user's
context seriously.**

---

## 3. The age parser that repaired instead of rejecting

```python
digits = "".join(ch for ch in answer if ch.isdigit())
```

| Input | Recorded as |
|---|---|
| `9.5` | **95** |
| `-5` | **5** |
| `²` | crash — `isdigit()` accepts it, `int()` refuses |

Nothing in the chat shows the recorded age, so a worker screened on a number
they never typed had no way to see it.

### The fix, and the subtlety

`answer.strip().isdecimal()` then the range check — **not** bare `int()`.

`isdecimal()` rejects `²`, `-5`, and `3_4`, while still accepting Devanagari
`३४` and Arabic-Indic `٣٤`, which is what this bot's users actually type. A
stricter ASCII-only parser would have broken the primary audience.

### The lesson

**Reject the whole input; never repair it.** Silent repair produces a confident
wrong answer, which is the one output this project exists to prevent.

---

## 4. The six from the first hour of real use

The first real testing session found six bugs at once. All in the conversation
layer:

- `reply_markup: null` → a silent HTTP 400 that killed every session
- a free-text occupation loop with no exit
- no "not working" category, no Hinglish keywords
- no zero-income band — and PM-SYM's list would have denied it
- f-string lookups missed the language, so income and land **button labels**
  stayed Hindi in English mode
- the pack used `name_hi` with hardcoded Hindi labels

### The lesson

Every failure was in the conversation layer, never the engine — and each test
that "passed" checked message text while the bug sat in button labels.

**Test the surface the user touches.**

---

## 5. The methodology bug real numbers exposed

An insurance **cover** (₹2,00,000, PMSBY) was being summed with an annual
**pension** (₹36,000, PM-SYM) into one "total benefit" figure — roughly a 6×
overstatement.

They are not the same unit. A cover is a contingent payout; a pension is
recurring income. Adding them produces a number that means nothing.

### The lesson

**Don't let one number stand in for two different things.** The bug only became
visible when real researched values replaced placeholders — placeholder data
hides unit errors.

---

## 6. A diagnostic that lied

`getUpdates` returning `200` was taken as proof that nothing else was polling.
It is not: Telegram **preempts** an existing long poll rather than always
returning `409`. That produced a confident "the service is not live" that was
wrong.

The reliable checks are `ss -tnp` showing an `ESTAB` connection to
`149.154.166.x:443` for the bot's PID, plus fresh `session_start` rows.

### The lesson

**Verify with the signal that cannot be faked.** A success code from an API you
misunderstand is not evidence.

---

## 7. The two blind tests

Covered in [07-testing.md](07-testing.md) — an acceptance test that compared
answer recaps instead of verdicts, and a walk that fabricated message ids and
tested every command against the opening screen.

### The lesson

**A green suite after a behaviour change is a claim, not evidence.** When a fix
invalidates a test's assumptions, repairing the test is part of the fix, not a
footnote.

---

## The pattern across all seven

| | |
|---|---|
| Where bugs were | conversation layer, adapter, tests |
| Where bugs were **not** | the rule engine |
| Most common shape | something looked verified/tested/correct without being it |
| Most common cause | a check that measured an adjacent thing |

The engine survived because it is small, pure, and has no I/O. Everything that
touches the outside world is where the defects live — which is an argument for
keeping the decision-making core as small as possible.

---

Next: [09-safety-and-privacy.md](09-safety-and-privacy.md)
