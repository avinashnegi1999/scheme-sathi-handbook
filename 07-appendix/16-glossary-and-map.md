# Glossary and file map

---

## Terms

**Unorganised worker** — a worker outside formal employment: no PF, no written
contract, no employer-provided social security. Roughly 90% of India's
workforce.

**e-Shram** — the national database of unorganised workers. Registering gives a
UAN. It is a **gateway**, not a payout, which is why its `annual_value_inr` is
₹0 and `value_basis = "gateway"`.

**PMSBY** — Pradhan Mantri Suraksha Bima Yojana. ₹2,00,000 accidental death and
disability cover for a ₹20/year premium.

**PM-SYM** — Pradhan Mantri Shram Yogi Maandhan. A pension of ₹3,000/month from
age 60, with matching monthly contributions during working years.

**CSC** — Common Service Centre. The physical office a worker walks to. The
place a wrong answer costs them a day's wages to reach.

**UAN** — Universal Account Number, the identifier e-Shram registration issues.

**myScheme** — `myScheme.gov.in`, the government's own scheme directory. This
project sits on top of it, not against it.

**Stub** — the literal string `"TODO"` in a scheme file. Means "nobody has
researched this yet", deliberately typed as a string even for numeric fields.

**Servable** — a scheme that is both researched and signed off by a human. The
only condition under which `evaluate()` will produce a verdict.

**Three-valued verdict** — `ELIGIBLE`, `INELIGIBLE`, `UNKNOWN`. `UNKNOWN` is a
first-class answer, not an error.

**Pack** — the one-page HTML sheet a worker carries into the centre: their
answers, the schemes, the documents needed.

**k-anonymity** — an aggregate is suppressed unless at least *n* records share
it. Here, n=5.

---

## File map

```
yojana-sathi/
├── check.py                    one command runs everything
├── sathi/
│   ├── main.py                 entry point, argparse
│   ├── core/
│   │   ├── profile.py          the ONLY data the engine reads — no PII fields
│   │   ├── schemes.py          load + validate data/schemes/*.toml
│   │   └── content.py          occupations, states, UI strings
│   ├── rules/                  ◀── NO LLM MAY BE IMPORTED HERE
│   │   ├── engine.py           evaluate() — the only place eligibility is decided
│   │   └── operators.py        the seven comparison operators
│   ├── conversation/
│   │   ├── consent.py          asked first, before anything is recorded
│   │   └── flow.py             the intake state machine
│   ├── channels/
│   │   ├── base.py             Reply / ChannelMessage — channel-agnostic
│   │   └── telegram.py         thin adapter: keyboards, callbacks, message ids
│   ├── render/
│   │   ├── templates.py        Results → Hindi a worker can hear
│   │   ├── llm.py              optional rephrasing, outside the engine
│   │   └── audio.py            optional Hindi audio notes
│   ├── pack/
│   │   ├── checklist.py        documents needed vs documents held
│   │   └── pack.py             the one-page sheet
│   └── metrics/
│       ├── schema.sql          events + followups
│       ├── events.py           the ONLY writer to the event DB
│       └── report.py           SQLite in, one HTML file out
├── data/
│   ├── schemes/*.toml          the welfare rules — data, not code
│   ├── occupations.toml
│   ├── states.toml
│   └── strings_{hi,en}.toml    124 keys, must match exactly
├── tests/                      6 files, plain asserts
├── deploy/
│   ├── provision-aws.sh        refuses to build a second instance
│   ├── install-on-vm.sh
│   ├── sathi.service
│   └── RUNBOOK.md
└── docs/
    ├── ARCHITECTURE.md
    ├── BUILD_LOG.md            every bug, unedited
    ├── LESSONS.md              ten lessons
    ├── IMPACT.md
    ├── SCHEME_AUTHORING.md
    └── VERIFICATION.md
```

---

## Where to look when

| Question | File |
|---|---|
| How is eligibility decided? | `sathi/rules/engine.py` |
| What can a scheme file say? | `sathi/rules/operators.py` |
| What questions are asked, in what order? | `sathi/conversation/flow.py` |
| Why did the bot send that exact message? | `data/strings_hi.toml` / `_en.toml` |
| What is stored about a worker? | `sathi/metrics/schema.sql` |
| What went wrong historically? | `docs/BUILD_LOG.md` |
| How do I deploy or roll back? | `deploy/RUNBOOK.md` |

---

## Reading the comment tags

The source uses Better Comments conventions:

| Tag | Meaning |
|---|---|
| `# !` | important — usually a trap, an invariant, or a bug that was fixed here |
| `# *` | a highlighted explanation of a non-obvious mechanism |
| `# ?` | an open question the author has not resolved |
| `# TODO` | outstanding work |

`# !` comments are worth reading on their own — most of them mark the site of a
real bug and explain why the code looks the way it does.

---

**End of handbook.** Back to [the start](../README.md).
