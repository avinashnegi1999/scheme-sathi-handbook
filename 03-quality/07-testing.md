# Testing

---

## What exists

| | |
|---|---|
| Command | `python3 check.py` |
| Module self-checks | 15, run in dependency order |
| Test files | 6 |
| Path coverage | 486 button-by-button paths **per language** |
| Framework | none — plain `assert`, nothing to install |
| Runtime | a few seconds |

Self-checks run **in dependency order**, so the first failure points at the
lowest broken layer rather than the five things downstream of it.

---

## The one lesson worth taking from this project's tests

> **A green suite is a claim, not evidence.**

This suite has been green while blind **twice**. Both times it was caught by
something other than the suite itself, and both are worth being able to tell.

---

### Blind #1 — the test that compared the wrong thing

`test_llm_path_and_button_path_agree` asserted:

```python
assert got == expected, "the LLM changed the result — it must only change phrasing"
```

Both `got` and `expected` were `reply[0].text`. But `reply[0]` is the **answer
recap**, which is built from the profile and is identical whatever the engine
decided. `reply[1]` is the verdict.

The test never looked at the result it claimed to protect. Replacing the verdict
with a sentinel string still passed.

**Fix:** compare every reply, not `reply[0]`, so a message inserted at the front
cannot blind it again.

---

### Blind #2 — the walk that stopped walking

Adding the stale-keyboard guard broke `test_all_paths.py` invisibly. The test
fabricated `message_id: i + 1` instead of using the id the wire returned, so
under the new guard **all 12 taps were rejected and no session was ever
created**. Every command was being tested against the opening screen.

The suite still printed `all checks passed`.

**Fix:** start at `/start`, tap the id the stub actually returned, type when no
keyboard is on screen, and answer NPS so the walk reaches `DOCUMENTS`, `PACK`
and `DONE` instead of stalling.

---

## The defence: tests that fail when they stop testing

After the six-bug session, the walk was given **coverage counters it asserts on
itself**:

```python
assert CHECKED["packs"] >= 2, f"no pack was ever generated or checked: {CHECKED}"
```

That assertion paid for itself almost immediately. Enforcing the verification
gate meant no scheme was servable, so no worker reached an eligible result, so
no pack was ever generated — and the walk quietly fell to a fifth of its paths
while still passing every assertion about the screens it *did* reach.

The counter caught it.

> **The principle:** a test that silently stops reaching the thing it checks is
> worse than no test, because it produces confidence instead of doubt. Make it
> assert its own reach.

---

## Test what the user touches

The most repeated failure in this project's history:

> Tests passed because they asserted on **message text** while the bug sat in
> **button labels**.

An English-language run once kept Hindi income and land buttons because the
f-string lookups missed the language — and every test passed, because they all
checked the message body.

The walk now checks button labels, reached states, generated packs, event-log
rows, and result messages. Not just the prose.

---

## Mutation testing, informally

Not a framework — just a habit. When a test is added for a bug, break the fix on
purpose and confirm the test fails. If it still passes, the test is decorative.

Both blind tests above were confirmed this way: corrupt the verdict, confirm the
old assertion passes and the new one fails.

---

Next: [08-the-bugs.md](08-the-bugs.md)
