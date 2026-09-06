# Scorecard

A scored breakdown with the evidence for each number. The point of writing it
down is the low score, not the high ones — a scorecard that flatters everything
says nothing.

---

| Dimension | Score |
|---|---|
| Idea / problem | **9** / 10 |
| Architecture | **9** / 10 |
| Testing | **9.5** / 10 |
| Safety / correctness | **9.5** / 10 |
| Documentation | **9** / 10 |
| Code / engineering | **9** / 10 |
| Current product maturity | **7** / 10 |
| Portfolio / interview value | **9** / 10 |

---

## Idea / problem — 9

A real, cited problem with a quantified cost of failure: ₹455/day male, ₹315/day
female casual labourer. Positioned as last-mile delivery on top of
`myScheme.gov.in` rather than a replacement, which shows the existing solution
was checked first.

> **Why not 10:** the problem is real but the demand is unproven. Nobody outside
> the build has used this.

## Architecture — 9

The eligibility boundary is structural, not conventional — `sathi/rules/` cannot
import a language model. Channel layer separable so Meta verification could
never block a deploy date. `LLM_API_KEY` unset is a *tested* configuration.

> **Why not 10:** the WhatsApp adapter is designed and unbuilt, so
> "channel-agnostic" is argued rather than demonstrated.

## Testing — 9.5

4,140 lines of production Python against 1,716 of tests. 15 module self-checks,
6 test files, 486 paths per language, one command, no framework.

> **Why the 0.5 off:** this suite has twice been green while blind — an
> acceptance test comparing recaps instead of verdicts, and a walk that
> fabricated message ids. Both fixed, both now fail if the bug returns. The
> lesson stands: **a green suite is a claim, not evidence.**

## Safety / correctness — 9.5

Three-valued verdicts. `"TODO"` over `0`. No PII fields in the schema at all.
Deep-linked provenance per value. No scheme served until a human signs it.

> **Why the 0.5 off:** correctness is **enforced, not proven**. The values are
> researched but unconfirmed, and one documented limitation remains — after a
> transient Telegram failure the worker's buttons go dead.

## Documentation — 9

Seven documents plus a deploy runbook with rollback. 732 comment lines written
to explain the non-obvious mechanism rather than restate the function name.
`BUILD_LOG.md` records the wrong turns, not a cleaned-up narrative.

> **Why not 10:** none of it has been read by someone who did not build it.

## Code / engineering — 9

Zero third-party dependencies. Hand-written multipart upload, Telegram client,
and rule engine. Docker builds and runs the full suite at image build time.

> **Why not 10:** `telegram.py` and `flow.py` are the two largest modules and
> carry most of the project's complexity.

---

## Current product maturity — 7

**The honest number, and the reason the rest is worth reading.**

The engineering is ahead of the product. What is missing is not code:

- No scheme value is signed off, so the bot correctly answers `UNKNOWN` for
  everything. **It cannot yet do the thing it was built to do.**
- The Hindi and English strings have never been read by a native speaker.
- Three schemes. National coverage is far larger.
- No real worker has completed a screening.
- The WhatsApp adapter and the follow-up sender are designed and unbuilt.

Every one of those is blocked on a **person** — a sign-off, a language review, a
recruited user — not on a missing function. That is the correct place for a
project like this to be blocked, and it is still a 7.

---

## Portfolio / interview value — 9

The defensible parts are the judgement calls, not the line count: refusing to
let a model near an eligibility decision, `"TODO"` over `0`, refusing to serve a
value no human confirmed, and a privacy model that makes the wrong thing
structurally impossible rather than merely forbidden.

Plus a real debugging record — a double tap that denied someone a pension, an
age parser that turned `9.5` into 95, a test suite that was green and blind.
Found, traced, fixed, covered by regressions that fail without the fix.

> **Why not 10:** until someone who is not the author has used it end to end.

---

## What would move the numbers

Maturity 7 → 9 needs **no new code**: sign off the scheme values, get the Hindi
reviewed by a native speaker, put it in front of real workers.

Most of the other scores move on the same evidence — an outside reader, an
outside user.

---

Next: [12-whats-unfinished.md](12-whats-unfinished.md)
