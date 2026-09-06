# The story

How to explain this project to someone in under two minutes, and the history
behind it that is worth knowing.

---

## The one-minute version

> Scheme Sathi is a Telegram agent that screens an unorganised worker against a
> deterministic rule engine and tells them, in spoken Hindi, which government
> welfare schemes they qualify for, what each is worth in rupees, and where to
> go to claim it.
>
> The interesting constraint is that the user is deciding whether to spend a day
> walking to a government centre. A male casual labourer earns ₹455 a day, a
> female ₹315. That is the price of a wrong answer, and it is what the whole
> architecture is built around.

If they only want thirty seconds, stop there.

---

## The chain worth reciting

This is the part that makes it an engineering project rather than a chatbot —
a wage figure becoming a type signature:

```
₹455/day is the cost of a wrong answer
        ↓
false positives and false negatives have different units
        ↓
a probabilistic component cannot produce the verdict
        ↓
the language model is banned from the rule engine
        ↓
what a rule cannot decide must be representable
        ↓
three-valued verdicts: ELIGIBLE | INELIGIBLE | UNKNOWN
        ↓
an unverified scheme must return UNKNOWN, never INELIGIBLE
```

Each arrow is forced. That is why the design is defensible rather than
preferred.

---

## Before this: a project that was parked

Scheme Sathi is the second entry for this hackathon. The first was **Saans**, a
personal air-exposure agent on Telegram — ~3,500 lines, 13 modules, zero
dependencies, 12/12 checks, a live bot, a public repo. Finished code, parked on
30 August 2026.

### How that track was picked

Not by interest. 50% of the hackathon score is a cited written problem statement
judged on size and severity, so tracks were ranked by citable headcount and
death toll. Clean Air — 1.4 billion exposed, 1.67 million deaths a year — beat
Tech Employment's 1.5 million graduates a year by roughly 18 points before a
line of code existed.

### Why it was parked anyway

It was never deployed and never put in front of a single user.

That is the real reason, and being precise about it matters, because the first
explanation given was a different one: the idea already existed in another app.
But novelty is not a judging criterion here.

> Losing on novelty was a feeling. Losing on *"25% deployment and impact data,
> ties broken on impact first, and you have zero users"* is a score.

**This is a strong thing to be able to say in an interview.** It shows the
ability to separate an emotional reason from a real one, and to act on the real
one.

The lesson carried into Scheme Sathi was to front-load deployment: the deploy
gate sits at week 6, because the impact criterion needs ~5 weeks of real usage
and cannot be retrofitted at the end.

---

## The AI question

Used heavily, documented openly in `LESSONS.md` and `BUILD_LOG.md`. The
disclosure is public, so being cagey would be both dishonest and pointless.

The defensible claim is not "I typed every line". It is that **the judgement
calls are mine and every one can be defended**:

- keeping the model out of the rule engine
- `"TODO"` over `0`
- refusing to serve a value no human signed
- a schema in which PII cannot exist

And there is evidence of *disagreeing* with review, which is the harder proof:
two review rounds asked for location-based routing while praising the privacy
model that forbids it, and several "findings" turned out to be the project's own
code comments read back.

> Being able to say *"I was told to do this and I didn't, here's why"* is worth
> more than any claim about authorship.

---

Next: [14-drill.md](14-drill.md)
