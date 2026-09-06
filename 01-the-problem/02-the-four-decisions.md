# The four decisions

Made before any code was written, each forced by the cost asymmetry in the
previous chapter. Every one of them has a consequence, and the consequence is
the part worth saying out loud — a rule without its consequence sounds like
dogma.

---

## 1. The language model may not touch an eligibility decision

`sathi/rules/` cannot import an LLM. Structurally, not by convention.

The model is allowed to: run conversation flow, map free text to a category
(always confirmed back to the worker before it is recorded), and rephrase Hindi
that a human wrote. It never sees a threshold, never produces a rupee figure,
never produces a verdict.

The shorthand is **compute, then narrate.**

> **Consequence:** the same profile always produces the same answer, and the
> exact line of the rule that produced it can be pointed at. A model that is
> right 95% of the time is wrong once in twenty — for someone spending a day's
> wages on the answer.

A stronger follow-up, if pushed: `LLM_API_KEY` unset is a **tested**
configuration, not a degraded one. Every eligibility result is identical with or
without it. The model is genuinely optional, which is the only way to prove it
isn't load-bearing.

## 2. An unresearched value is the string `"TODO"`, never `0`

Including for numeric fields. `annual_value_inr = "TODO"`.

> **Consequence:** a zero looks researched. A validator cannot distinguish "this
> scheme is worth ₹0" from "nobody has looked this up yet". A string breaks
> loudly the moment anything tries to do arithmetic on it — which is exactly the
> behaviour wanted.

This is a general principle worth stealing: **make the unfinished state
un-representable as a valid value.** A sentinel that type-checks is a bug
waiting for a quiet afternoon.

## 3. Verdicts are three-valued

`ELIGIBLE`, `INELIGIBLE`, `UNKNOWN`.

`UNKNOWN` is returned when a profile field is missing, *or* the scheme file still
carries a `"TODO"` the rule needs, *or* no human has signed the file off. A gap
is never filled with a default.

> **Consequence:** the system can say "I have not checked this", which is the one
> thing a confidently wrong answer cannot do. And `UNKNOWN` is not a dead end —
> it ships with a question the worker can ask at the centre, so they still leave
> with something usable.

## 4. The profile has no name, phone, or Aadhaar field

Not "we don't store it" — the field does not exist in the schema.

```
Profile: state, age, occupation, income_band, land_holding_band, family_size,
         has_bank_account, is_income_tax_payer, is_epfo_or_esic_member,
         is_nps_member, known_schemes
```

> **Consequence:** a field that does not exist cannot be leaked by a future bug,
> a careless log line, or a database dump. Prose in a privacy policy is a
> promise. A missing column is a guarantee.

The event log holds coarse bands only, under a random per-session id that is
**not** derived from the Telegram id, with k-anonymity suppression below n=5.

---

## The pattern across all four

Each decision moves a safety property from something that must be *remembered*
to something that is *structurally true*. The LLM can't reach the rules because
of an import boundary. A stub can't be mistaken for data because of its type. A
worker's name can't leak because there is no field. Discipline is fragile;
structure is not.

Next: [../02-how-it-works/03-architecture.md](../02-how-it-works/03-architecture.md)
