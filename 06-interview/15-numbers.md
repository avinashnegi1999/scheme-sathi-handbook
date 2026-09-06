# Numbers to know cold

Every figure here was checked against the repo. If a number is not on this
page, do not quote it.

---

## The project

| | |
|---|---|
| Cost of a wrong answer | ₹455/day male, ₹315/day female casual labourer |
| Schemes shipped | **3** — e-Shram, PMSBY, PM-SYM |
| Scheme files in `data/schemes/` | 4 — the fourth is `_TEMPLATE.toml`, not a scheme |
| Verdict states | 3 — `ELIGIBLE`, `INELIGIBLE`, `UNKNOWN` |
| Operators a scheme may use | 7 — `exists`, `between`, `gte`, `lte`, `in`, `not_in`, `eq` |

## The code

| | |
|---|---|
| Production Python | 4,140 lines |
| Test Python | 1,716 lines |
| Comment lines | 732 |
| Third-party dependencies | **zero** — stdlib only |
| Python required | 3.11+ (the VM runs 3.12) |
| Largest modules | `channels/telegram.py`, `conversation/flow.py` |

## Testing

| | |
|---|---|
| Module self-checks | 15 |
| Test files | 6 |
| Paths walked | **486 per language** |
| Command | `python3 check.py` |
| Framework | none |

## Deployment

| | |
|---|---|
| Host | AWS EC2, `ap-south-1` |
| Path | `/opt/sathi` — a file copy, **not** a git checkout |
| Service | systemd unit `sathi` |
| Cost | ~$10.50/month against $120 credit |

## Scheme values

| Scheme | Benefit | Key rule |
|---|---|---|
| PMSBY | ₹2,00,000 accident cover, ₹20/yr | age 18–70, bank account |
| PM-SYM | ₹3,000/month pension from 60 | age 18–40, income ≤ ₹15,000/mo, excludes EPFO/ESIC/NPS and tax payers |
| e-Shram | a UAN — **₹0 of its own** | age 16+ — it is a gateway, not a payout |

> e-Shram being worth ₹0 is not a gap. It is a registration gateway, and
> recording that honestly as `value_basis = "gateway"` rather than inventing a
> figure is the correct behaviour.

---

## Numbers NOT to quote

These have appeared in AI-generated summaries of this project and are **wrong**:

| Claim | Reality |
|---|---|
| "954 test paths" | 486 per language |
| "954 state-age-gender combinations" | there is no gender field; tests use one age and one state |
| "44M+ unorganized workers" | not in the repo — unsourced |
| "zero-persistence architecture" | false — `sathi.db` has two tables |
| "guarantees 0% hallucination" | the model is optional and off; that is a design choice, not a proof |
| "works without a smartphone" | Telegram requires one |
| "PM-KISAN", "Ayushman Bharat" | not in this project |
| `engine/core.py` | the path is `sathi/rules/engine.py` |

> Quoting an unverifiable number on a résumé is the exact failure this project
> exists to prevent. An interviewer who opens `schema.sql` and finds two tables
> has caught a lie about the one thing the project claims to care about.

---

Next: [../07-appendix/16-glossary-and-map.md](../07-appendix/16-glossary-and-map.md)
