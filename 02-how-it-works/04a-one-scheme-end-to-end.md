# One scheme, end to end

The muscle-memory chapter. One real scheme, PMSBY, followed from its TOML file
to a verdict — with actual output, not a description of what would happen.

If an interviewer says *"open `engine.py` and walk me through it"*, this is the
page that makes that easy.

---

## Step 1 — the file

`data/schemes/pmsby.toml`, loaded by `core/schemes.py`:

```python
>>> from sathi.core.schemes import load_all
>>> s = load_all()["PMSBY"]
```

Its two criteria:

| field | op | value |
|---|---|---|
| `age` | `between` | `[18, 70]` |
| `has_bank_account` | `eq` | `True` |

No exclusions. Benefit: ₹2,00,000 accident cover for a ₹20/year premium.

---

## Step 2 — the three gate properties

```python
>>> s.is_researched      # no "TODO" left — somebody looked the values up
True
>>> s.is_human_verified  # verified_by carries a real signature
False
>>> s.is_servable        # both
False
>>> s.stubs
()
```

**Read those four lines carefully — they are the whole safety model.**

The file is fully researched. `stubs` is empty; there is no `TODO` anywhere in
it. Under the old `is_verified = not self.stubs` implementation, this scheme
would have passed the gate and served real verdicts.

It does not, because `verified_by` still reads
`unconfirmed — PENDING HUMAN VERIFICATION`.

---

## Step 3 — evaluate

```python
>>> from sathi.core.profile import Profile
>>> from sathi.rules import engine
>>> p = Profile(age=30, has_bank_account=True)
>>> r = engine.evaluate(p, s)
>>> r.verdict.name
'UNKNOWN'
>>> r.reasons[0].code
<ReasonCode.UNVERIFIED_DATA: 'unverified_data'>
>>> r.reasons[0].detail
"awaiting sign-off: verified_by = 'unconfirmed — PENDING HUMAN VERIFICATION'"
```

This worker is 30 and has a bank account. They **clearly** satisfy both
criteria. The engine returns `UNKNOWN` anyway.

> Precedence rule 1 fires first: a scheme that is not servable returns `UNKNOWN`
> **before any criterion is examined**. The age check never runs.

That is the gate doing its job, and it is the single best thing to be able to
demonstrate live.

---

## Step 4 — the operators, called directly

The criteria *would* have passed. Proof, bypassing the engine:

```python
>>> from sathi.rules import operators
>>> operators.apply("between", 30, [18, 70])
True
>>> operators.apply("eq", True, True)
True
```

So the `UNKNOWN` is not a criterion failing. It is the engine refusing to answer
from data no human has signed.

**This is the distinction to draw in an interview:** *"the rules pass — the
engine still won't answer, because passing rules on unverified numbers is
exactly how the last bug shipped."*

---

## Step 5 — what happens once it is signed

Signing the file in memory only, to show the other three paths:

```python
>>> signed = dataclasses.replace(s, verified_by="Avinash Negi, checked at source 2026-09-07")
>>> signed.is_servable
True
```

| Profile | Verdict | Why |
|---|---|---|
| age 30, bank yes | `ELIGIBLE` | both criteria pass |
| age 75, bank yes | `INELIGIBLE` | 75 is outside `[18, 70]` — a definite fail |
| age 30, bank **unanswered** | `UNKNOWN` | `eq(None, True)` returns `None` — undecidable |

The third row is the one worth pausing on. The worker is 30, so the age
criterion passes. But the bank question is unanswered, so `apply()` returns
`None`, which becomes precedence rule 4 — `UNKNOWN`, with the missing field
named so the bot can ask for it.

A missing answer does not become a `False`. **That is three-valued logic doing
real work**, not a stylistic choice.

---

## The whole thing in one paragraph

> PMSBY is fully researched — no `TODO` anywhere — but `verified_by` still says
> PENDING, so `is_servable` is False and `evaluate()` returns `UNKNOWN` before
> it looks at a single criterion. If I sign the file, the same worker comes back
> `ELIGIBLE`; a 75-year-old comes back `INELIGIBLE`; and a worker who skipped
> the bank question comes back `UNKNOWN` with the missing field named. The
> operators pass in all three cases — what changes is whether the engine is
> willing to speak.

Say that out loud until it is automatic. It covers the gate, the precedence
order, three-valued logic, and the bug that caused all of it.

---

## Try it yourself

```bash
cd "/run/media/avinash/Data/project Scheme Sathi"
python3 -c "
import sys; sys.path.insert(0,'.')
from sathi.core.schemes import load_all
from sathi.core.profile import Profile
from sathi.rules import engine
s = load_all()['PMSBY']
print('servable:', s.is_servable)
print(engine.evaluate(Profile(age=30, has_bank_account=True), s).verdict.name)
"
```

Ten seconds, and the abstract becomes concrete.

---

Next: [05-scheme-files.md](05-scheme-files.md)
