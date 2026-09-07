# Scheme Sathi — the handbook

Everything about how [Scheme Sathi](https://github.com/avinashnegi1999/yojana-sathi)
was built, why it is shaped the way it is, and what I should be able to say
about it. Written to be read from the first page to the last.

This is a companion to the code, not a copy of it. The repo has the source and
the build log; this has the explanation.

---

## How to read this

**Straight through, in order.** Each part assumes the one before it. If you only
have twenty minutes, read `01-the-problem/` and stop — the rest will not make
sense without it, and it is the half that matters most in an interview.

| Part | What it covers | Read it when |
|---|---|---|
| [01 — The problem](01-the-problem/) | Who this is for, what is actually broken, why the design is forced | First. Always first |
| [02 — How it works](02-how-it-works/) | Architecture, the rule engine, **one scheme traced end to end**, scheme files, conversation, Telegram | You want the technical picture |
| [03 — Quality](03-quality/) | Testing, every bug and what it taught, the safety model | You want to know whether to trust it |
| [04 — Operations](04-operations/) | Deployment, the live host, what to do when it breaks | You need to run or fix it |
| [05 — Assessment](05-assessment/) | Scored breakdown with evidence, and what is unfinished | You want the honest state |
| [06 — Interview](06-interview/) | The story, a self-test, numbers to know cold | You have an interview |
| [07 — Appendix](07-appendix/) | Glossary, file map | You hit a term or a filename you do not recognise |
| [08 — Video](08-video/) | The 10-minute build story in Hinglish and English, and the 3-minute demo narration | You are recording, or you want the story in spoken form |

## The shortest possible summary

A Telegram agent that screens an unorganised worker against a deterministic rule
engine and tells them, in spoken Hindi, which government welfare schemes they
qualify for, what each is worth in rupees, and where to go to claim it.

The constraint that shapes everything: the user is deciding whether to spend a
day walking to a government centre. A male casual labourer earns ₹455 a day, a
female ₹315. That is the price of a wrong answer.

## A note on honesty

Two things in here are uncomfortable and stay in on purpose.

The bot currently answers `UNKNOWN` for every scheme, because no value has been
signed off by a human yet. That is the safety gate working, not a broken build.

And the test suite has twice been green while blind. Both times are written up
in [03-quality/08-the-bugs.md](03-quality/08-the-bugs.md), because a handbook
that only records the wins is not worth reading.
