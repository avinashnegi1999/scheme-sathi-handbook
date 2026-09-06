# Scheme files

Where the welfare rules live. TOML, not Python — the person who understands
welfare policy should not need to understand code.

---

## What one looks like

```toml
code          = "PM_SYM"
name_hi       = "प्रधानमंत्री श्रम योगी मानधन"
authority     = "Ministry of Labour and Employment"
official_url  = "https://maandhan.in/..."
verified_on   = "2026-08-31"
verified_by   = "unconfirmed — PENDING HUMAN VERIFICATION"

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

Every value carries a **deep-linked `source_url`** — the specific page or
anchor that proves it, never the site root — plus a `verified_on` date.

---

## The three states a scheme file can be in

This distinction was learned the hard way and is the most useful idea in the
data model.

| State | Meaning | Effect |
|---|---|---|
| **STRUCTURAL** | unknown key, bad operator, unknown `Profile` field | `SchemeError` — the app refuses to start |
| **UNRESEARCHED** | a `"TODO"` remains | loads, flagged in `Scheme.stubs` |
| **UNSIGNED** | `verified_by` still says `PENDING HUMAN VERIFICATION` | loads, `is_human_verified` is False |

> A structural problem is a **programmer's** bug and should crash loudly at
> startup. An unresearched or unsigned file is a **process** state and should
> load, be visible, and simply refuse to serve verdicts.
>
> Conflating those two was the verification bug.

---

## Why `"TODO"` and not `0` or `None`

`annual_value_inr = "TODO"` — a string, in a numeric field, on purpose.

- `0` looks researched. A validator cannot tell "worth nothing" from "not looked up".
- `None` type-checks in Python and slips through arithmetic as a silent failure later.
- `"TODO"` breaks loudly the moment anything tries to compute with it.

The general principle: **make the unfinished state un-representable as a valid
value.** A sentinel that type-checks is a bug waiting for a quiet afternoon.

`_is_stub()` in `operators.py` checks for it before any comparison and returns
`None`, which becomes `UNKNOWN` at the top.

---

## Adding a scheme

No Python required. Copy `data/schemes/_TEMPLATE.toml`, fill it from official
sources, cite every value.

**The repo's contribution rule:** *"Contributions without a `source_url` and
`verified_on` per value are closed."*

That is a strong statement to be able to point at — it means the provenance
requirement is enforced socially as well as structurally.

---

## The interview question this sets up

> *"The government changes a benefit amount. What do you edit?"*

`data/schemes/pm_sym.toml` — change the field, update `source_url` to the deep
link that proves the new number, set `verified_on` to today, and re-sign
`verified_by`. **No Python is touched**; the engine reads the file at startup.

That answer demonstrates maintainability thinking, which is what separates
someone who ships from someone who codes.

---

## What is actually shipped today

Three schemes, all unsigned:

| Scheme | What it is | Status |
|---|---|---|
| e-Shram | registration gateway — a UAN, not a payout | `PENDING HUMAN VERIFICATION` |
| PMSBY | ₹2,00,000 accident cover, ₹20/yr premium | `PENDING HUMAN VERIFICATION` |
| PM-SYM | ₹3,000/month pension from age 60 | `PENDING HUMAN VERIFICATION` |

Plus `_TEMPLATE.toml`, which is a template and not a scheme — worth remembering,
because "four files" and "three schemes" are both true and only one is the right
thing to say.

---

Next: [06-conversation-and-telegram.md](06-conversation-and-telegram.md)
