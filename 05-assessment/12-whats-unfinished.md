# What is unfinished

Kept separate from the scorecard so it can be read as a list of work rather
than a list of excuses. Nothing here is blocked on writing code.

---

## 1. Scheme sign-off — the blocker

Three schemes are researched from official sources and deep-linked. None is
signed. Until each value is confirmed at source and `verified_by` carries a real
name, every worker receives `UNKNOWN`.

**This is the gate working.** It is also the single thing standing between the
project and being usable.

> There is one open research question: the PMSBY age cap reached *through* the
> e-Shram route. e-Shram itself has no upper age bound in the visible FAQ, but a
> `59` appears inside an unclosed HTML comment in Question 40 and caps PMSBY via
> that route. A 65-year-old may be told they get PMSBY cover when they do not.
> **That needs a phone call to a CSC operator, not another reading of the FAQ.**

## 2. Native Hindi review

`data/strings_hi.toml` and `strings_en.toml` were drafted without a native
speaker reading either. The pipeline can be built; the register cannot be
certified by the person who built it.

## 3. Real users

Nobody outside the build has completed a screening. This is the largest gap and
no amount of engineering closes it.

## 4. Coverage

Three schemes. e-Shram, PMSBY, PM-SYM. The authoring path is documented so
adding more is research effort, not engineering effort.

## 5. Built for, not built

- WhatsApp adapter — designed for, deliberately unbuilt
- Follow-up sender — storage and purge exist, the sender does not
- PDF packs — HTML instead, because the stdlib has no PDF writer

---

## The known technical limitation

After a transient Telegram failure the worker's buttons go dead. Restoring the
keyboard would let the old question's buttons answer the new state — the exact
bug the message-id guard exists to prevent.

A real fix means advancing the `getUpdates` offset only after a turn commits,
which risks double-writing events. **Documented in the code, deliberately not
patched over.**

---

## What this list says about the project

Every item is a person-shaped blocker: a sign-off, a review, a recruited user, a
phone call. None is "the code doesn't work".

For a portfolio piece that is a good place to be stuck, and it is worth saying
plainly rather than apologising for.

---

Next: [../06-interview/13-the-story.md](../06-interview/13-the-story.md)
