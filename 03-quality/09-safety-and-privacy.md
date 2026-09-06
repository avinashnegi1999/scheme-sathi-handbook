# Safety and privacy

---

## The design rule

> Collect less, so there is less to protect.

Privacy here is not a policy document. It is an absence of fields.

---

## What the profile can hold

```
state              age                  occupation
income_band        land_holding_band    family_size
has_bank_account   is_income_tax_payer  is_epfo_or_esic_member
is_nps_member      known_schemes
```

**There is no name field. No phone field. No Aadhaar field.**

Not "we choose not to store them" — they do not exist in the dataclass. A field
that does not exist cannot be leaked by a future bug, a careless log line, or a
database dump.

> Prose in a privacy policy is a promise.
> A missing column is a guarantee.

Note also what the bands do: `income_band`, not income. `family_size`, not
household composition. Even the data that *is* collected is deliberately coarse.

---

## What is stored

`sathi.db` — SQLite, two tables: `events` and `followups`.

**Do not say "zero persistence".** It is false and `schema.sql` is one click
away. The accurate claim is stronger:

| Property | Detail |
|---|---|
| Granularity | coarse bands only, never raw answers |
| Identity | random per-session id, **not** derived from the Telegram id |
| Aggregation | k-anonymity suppression below n=5 |
| Writer | `metrics/events.py` — the only module that may write |

A single writer matters: it means the privacy invariants are enforced in one
file that can be read in full, not scattered across every call site.

---

## Consent comes first

`conversation/consent.py` runs **before anything is recorded**. Declining is a
real path — it stores nothing and ends the conversation. There is a test for it:

```python
out = c3.handle(consent.NO)
assert out[0].end and c3.profile.age is None
```

---

## No auto-submission

The bot generates an application pack. **The human submits it.**

There is no code path that files anything with a government portal on a worker's
behalf. That is a deliberate boundary: an agent that can act irreversibly on
behalf of someone who cannot read the confirmation screen is a different and
much more dangerous product.

---

## The safety gate

The strongest safety property, and the one currently most visible:

> No scheme value is served until a human signs it off.

All three schemes read `PENDING HUMAN VERIFICATION`, so every worker receives
`UNKNOWN` for all of them. A test enforces that string until the values are
checked by a person:

```
test_researched_is_not_the_same_as_signed_off
```

That test sits in a block marked, in the source:

> `# ! Never delete a test in this block to make a build pass.`

---

## What "safe" does and does not mean here

**It does mean:** the system cannot produce a confident verdict from unverified
data, cannot store identifying information, and cannot act on a worker's behalf.

**It does not mean:** the scheme values are correct. They are researched from
official sources and deep-linked, but not yet confirmed by a person. Correctness
here is **enforced, not proven** — the gate guarantees that unverified data
cannot be served, not that the data is right.

That distinction is worth stating precisely in an interview. It is the
difference between a safety *mechanism* and a safety *claim*.

---

Next: [../04-operations/10-running-it.md](../04-operations/10-running-it.md)
