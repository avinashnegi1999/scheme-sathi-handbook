# The conversation layer and the Telegram adapter

Where every bug in this project has lived. Not one has ever been in the engine.

---

## The split

| | `conversation/flow.py` | `channels/telegram.py` |
|---|---|---|
| Knows about | states, questions, answers | message ids, keyboards, callbacks |
| Does not know about | Telegram | eligibility, or what a scheme is |
| Size | 690 lines | 622 lines |

The adapter is deliberately thin: it translates a `Reply` into an API call and
an incoming update into an answer string. It holds no logic about what any
answer *means*.

---

## The state machine

`Conversation.handle()` is four lines and dispatches entirely by name:

```python
def handle(self, answer: str) -> list[Reply]:
    answer = (answer or "").strip()
    handler = getattr(self, f"_on_{self.state.value}", None)
    if handler is None:
        return [Reply(text=self._s("errors.generic"), end=True)]
    return handler(answer)
```

Each `_on_<state>` method validates the answer, records it, sets the next state,
and returns replies.

### The happy path, in order

```
LANGUAGE → CONSENT → STATE → AGE → OCCUPATION → INCOME → LAND
   → FAMILY → BANK → TAX → [TAX_CONFIRM] → EPFO_ESIC → NPS
   → KNOWN_SCHEMES → DOCUMENTS → PACK → DONE
```

**Be able to say this out loud.** It is the single most likely interview
question about the code.

---

## Why dispatch-by-name matters — and what it cost

Routing on `self.state` alone is clean and short. It also caused the worst
runtime bug in the project.

Telegram leaves old inline keyboards **live and tappable forever**. The callback
payload was a bare `"yes"` carrying no question identity. So:

1. Worker taps **Yes** on *"Do you have a bank account?"* → recorded, state → `TAX`
2. Nothing appears to happen (slow network), so they tap **Yes** again
3. `handle()` routes on the *current* state, `TAX`
4. They are now recorded as an income tax payer

PM-SYM excludes income tax payers. **A double tap silently denied a worker a
₹3,000/month pension, and the reply looked completely normal.**

On a cheap phone over weak rural 4G, tapping again because nothing happened is
not misuse — it is the expected behaviour of the target user.

### The fix

The adapter now remembers the `message_id` of the keyboard it last delivered and
accepts a callback only from that message, retiring it **before** dispatch.

Retiring *before* rather than after matters: multi-select screens like
`KNOWN_SCHEMES` and `DOCUMENTS` stay in the same state across taps, so a
state-based guard would let a duplicate tap toggle a selection back off.

---

## Three-valued answers reach the conversation too

The tax question offers **Yes / No / Don't know**.

`Don't know` is not stored as `False`. It leaves the field unset, so any scheme
excluding tax payers returns `UNKNOWN` with *"ask this at the centre"* attached.

The same idea as the engine, one layer up: a person who does not know is not the
same as a person who said no.

---

## The tax read-back

A `Yes` to the income tax question now shows both answers back:

```
You selected:
Monthly income: Up to ₹5,000
Income tax: Yes

Keep these answers?
[ Keep both ]  [ Change income ]  [ Change tax answer ]
```

**Note what it does not do.** It does not claim the answers contradict each
other. "Earns under ₹5,000" and "pays income tax" are only incompatible if you
know India's exemption limit — and importing that number would be an
unresearched value, exactly what this project refuses to ship.

So it asserts nothing, names no threshold, and takes the worker at their word if
they confirm. It exists purely because that one answer silently excludes someone
from PM-SYM, so a mis-tap there is expensive.

---

## Failure handling

A failure inside one conversation must never take the bot down for everyone
else. But the split matters:

| Failure | Response |
|---|---|
| 429, 5xx, API unreachable | re-raise → exponential backoff, **session preserved** |
| 400, 403, a bug in our handling | abandon the turn, tell the worker to `/start` again |

A rate limit says Telegram is busy — it says nothing about the worker's
conversation, and must not cost them their answers. A 400 means we sent
something malformed, which leaves the turn half-applied and unsafe to continue.

> **Known limitation:** after a transient failure the worker's buttons go dead.
> Restoring the keyboard would let the old question's buttons answer the new
> state — the exact bug the guard exists to prevent. Documented, not hidden.

---

Next: [../03-quality/07-testing.md](../03-quality/07-testing.md)
