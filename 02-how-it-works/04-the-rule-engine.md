# The rule engine

The most important 264 lines in the project, and the file an interviewer is
most likely to open.

---

## `evaluate(profile, scheme) -> Result`

A pure function. One worker, one scheme, one verdict. No I/O, no clock, no
randomness, no network — the same inputs always produce the same output.

### Precedence, in order

| # | Condition | Verdict |
|---|---|---|
| 1 | Scheme not servable | `UNKNOWN` — always, no further checking |
| 2 | Any exclusion definitely hits | `INELIGIBLE` |
| 3 | Any criterion definitely fails | `INELIGIBLE` |
| 4 | Anything undecidable | `UNKNOWN` |
| 5 | Otherwise | `ELIGIBLE` |

**Why this order.** A definite NO outranks a maybe, so rules 2 and 3 sit above
4 — if a worker is definitely excluded, a separate missing field doesn't make
that uncertain.

Rule 1 sits above everything because a scheme nobody has finished must not
produce a verdict *at all* — not even a negative one.

---

## `is_servable`, and the bug behind it

Three properties, deliberately separate:

```
is_researched      no "TODO" left — somebody looked the values up
is_human_verified  verified_by carries a real signature, not the PENDING marker
is_servable        both of the above
```

**`is_servable` is the only thing `evaluate()` is allowed to gate on.**

That separation exists because of the worst bug in the project's history.
`is_verified` was once implemented as:

```python
@property
def is_verified(self) -> bool:
    return not self.stubs
```

So "verified" actually meant "contains no TODO". All three scheme files had been
fully researched, so none had a `TODO` left — while every one still read
`verified_by = "unconfirmed — PENDING HUMAN VERIFICATION"`.

The result: startup printed `PMSBY: verified 2026-08-31`, and the engine served
real `ELIGIBLE` and `INELIGIBLE` verdicts off values one person had transcribed
from a PDF once. The README said in bold not to screen a real worker on this
data. The runtime ignored the README.

> The entire premise of the project is *don't guess*. The program was guessing
> that **researched** meant **verified** — which are not the same thing at all.

Full story: [../03-quality/08-the-bugs.md](../03-quality/08-the-bugs.md)

---

## Why an unservable scheme returns UNKNOWN and not INELIGIBLE

This is the subtlest decision in the codebase and a good interview answer.

An unverified scheme could go either way. Returning `INELIGIBLE` would be
convenient — it's a definite answer, it makes the demo cleaner, and the worker
simply doesn't hear about that scheme.

It is also the **expensive direction of the error**. Turning someone away on
data nobody checked costs them a scheme they may have been entitled to, silently
and invisibly, and they will never know to ask again.

`UNKNOWN` costs a conversation at the centre. `INELIGIBLE` costs an entitlement.

---

## The operators

Seven, in `rules/operators.py`. A scheme file may use only these:

| Operator | Meaning | Note |
|---|---|---|
| `exists` | is this field answered at all | the only one decidable with no answer |
| `between` | `[low, high]` | **inclusive at both ends** |
| `gte` / `lte` | numeric threshold | rejects bools — see below |
| `in` / `not_in` | membership in a list | |
| `eq` | exact match | no coercion: `"yes" != True`, `18 != "18"` |

A scheme file cannot express arbitrary logic — it cannot call code, branch, or
compute. That is deliberate. It means a scheme file is **data reviewable by
someone who does not program**, and the set of things a bad file can do is
bounded.

### Three details worth knowing

**`apply()` returns `True`, `False`, or `None`.** `None` means "cannot decide
from what we have" and is what propagates into precedence rule 4.

**Two ways to get `None`, checked before anything else:** the expected value is
still a `"TODO"` stub, or the worker's answer is missing. `exists` is the
exception — it is checked first, because for that operator absence *is* the
answer.

**`bool` is a subclass of `int` in Python.** So `True` would silently compare as
`1` against an age band. `_num()` rejects bools explicitly:

```python
if isinstance(v, bool) or not isinstance(v, (int, float)):
    raise OperatorError(f"{where}: expected a number, got {v!r}")
```

That raises rather than returning `None`, because it is a **scheme-file bug**,
not missing data. The distinction runs through the whole engine: uncertainty is
answered with `UNKNOWN`; malformed input is an error the caller reports and
never hides.

---

## Three-valued logic in practice

```
criterion: age between [18, 40]

profile.age = 30    →  True   → passes
profile.age = 55    →  False  → INELIGIBLE
profile.age = None  →  None   → UNKNOWN, with "we need your age" attached
```

The third row is the whole design. Most systems would default a missing age to
0, or to "fails", or would refuse to run. This one carries the uncertainty all
the way to the worker and turns it into a question they can ask.

---

Next: [05-scheme-files.md](05-scheme-files.md)
