# Who this is for, and what it costs them to be wrong

## The user

An unorganised worker in India. Construction labourer, domestic worker, street
vendor, farm hand. Roughly 90% of India's workforce. Typically: limited literacy,
a cheap Android phone, patchy 4G, and no idea which of ~3,000 government welfare
schemes apply to them.

They are not the user of a dashboard. They are someone standing in a lane
deciding whether tomorrow is a work day or a trip to a government office.

## The number that determines the architecture

A male casual labourer earns **₹455 a day**. A female, **₹315**.

That is a cited figure, not a rhetorical one, and it is the single most important
input to every design decision in this project. It converts an abstract question
("should the bot be careful?") into arithmetic:

- A wrong **yes** — telling someone they qualify when they don't — costs a day's
  wages plus the bus fare, and they usually do not come back for a second try.
- A wrong **no** — telling someone they don't qualify when they do — costs them
  a scheme they were entitled to, potentially for years.

**These are not the same error and they are not symmetric.** Most systems treat
false positives and false negatives as a tuning dial. Here they have different
units: one costs a day, the other costs years.

## Why that forces the design

Once you accept the asymmetry, several things stop being preferences:

1. If a wrong answer is this expensive, a probabilistic component cannot produce
   the answer. → the language model is banned from the rule engine.
2. If you cannot verify something, saying "I don't know" beats guessing in either
   direction. → three-valued verdicts, not boolean.
3. If a wrong *no* is the costly-but-invisible error, an unverified scheme must
   return `UNKNOWN` and **not** `INELIGIBLE`. → turning someone away on unchecked
   data is the direction you must never fail in.

That chain — from a wage figure to a type signature — is the thing worth being
able to recite. It is what makes this an engineering project rather than a
chatbot.

## What already exists, and why this isn't a duplicate

`myScheme.gov.in` is the government's own scheme directory. It is comprehensive
and well-built. It assumes literacy, a browser, and the ability to self-select
from thousands of entries.

Scheme Sathi is **last-mile delivery on top of it**, not a replacement. Always
say this. It demonstrates the existing solution was checked first, which is the
difference between solving a problem and noticing one.

Next: [02-the-four-decisions.md](02-the-four-decisions.md)
