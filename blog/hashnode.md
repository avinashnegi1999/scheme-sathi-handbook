# A Rule Engine That Isn't Allowed to Guess

### Building a welfare bot where a confident wrong answer can cost someone a day's wage

Most software fails by crashing. Yojana Sathi has a different problem: it can fail by giving someone a confident answer that turns out to be wrong.

The bot helps workers in India check government welfare schemes. It asks a few questions, checks the rules, and tells them which schemes they may qualify for, what the benefit is, and where they need to go.

But if the bot says **yes** when the real answer is **no**, the cost is not just a bad screen. Someone may take a day off work, pay for transport, wait at a government office, and then be turned away.

A male casual worker in India earned around **₹455 per day in 2025** on average. For a woman, it was around **₹315**.

So before thinking about features, I asked one question:

> **What is the wrong output that costs the user the most?**

For this project, the answer was simple:

> **A wrong “yes”.**

That decision shaped almost everything that came after.

---

## 1. The AI Cannot Decide Eligibility

The most important rule in the project is simple:

**The language model is not allowed to decide whether someone qualifies for a scheme.**

The eligibility code lives inside `sathi/rules/`. That package is not allowed to import an LLM SDK, HTTP client, or socket.

And this is enforced by a test.

```python
def test_engine_does_not_import_an_llm():
    code = (
        "import sys; import sathi.rules.engine;"
        "banned={'urllib.request','http.client','socket','ssl',"
        "'anthropic','openai','requests'};"
        "bad=[m for m in sys.modules "
        "if m in banned or 'llm' in m.lower()];"
        "print(','.join(sorted(bad)))"
    )

    out = subprocess.run(
        [sys.executable, "-c", code],
        cwd=ROOT,
        capture_output=True,
        text=True,
        check=True,
    ).stdout.strip()

    assert out == "", f"rules/ pulled in {out}"
```

The test runs in a subprocess because the main test process already has many imported modules.

The model still has a small role. It can help map free text into a category or rephrase Hindi text written by a human.

But it never:

* sees an eligibility threshold
* creates a rupee amount
* decides ELIGIBLE or INELIGIBLE

The whole flow also works with `LLM_API_KEY` missing.

That is intentional.

If the AI disappears, the eligibility result should stay exactly the same.

---

## 2. The Engine Has Three Answers

A normal Boolean gives you two values:

```text
True
False
```

That is not enough for this problem.

If a worker skips a question, `False` would be wrong. We simply do not know yet.

So the rule engine uses three states.

```python
def apply(
    op: str,
    actual: object,
    expected: object,
) -> bool | None:
    """Compare one profile value against one scheme rule."""

    if op == "exists":
        return actual is not None

    if _is_stub(expected):
        return None
```

The results mean:

| Result           | Meaning                          |
| ---------------- | -------------------------------- |
| `True` / `False` | The condition can be decided     |
| `None`           | The answer is genuinely unknown  |
| `OperatorError`  | The scheme rule itself is broken |

This lets the bot separate:

**“You do not qualify.”**

from:

**“I cannot decide yet.”**

Those are very different answers.

---

## 3. Python Has One Small Trap Here

Python treats `bool` as a subclass of `int`.

That means this is true:

```python
isinstance(True, int)
```

For normal programs, that may not matter much.

For an eligibility engine, I do not want `True` quietly becoming `1` in a numeric rule.

So numeric comparisons reject Boolean values directly.

```python
def _num(v: object, where: str) -> int | float:
    if isinstance(v, bool) or not isinstance(v, (int, float)):
        raise OperatorError(
            f"{where}: expected a number, got {v!r}"
        )

    return v
```

If the scheme file is broken, I want the program to complain.

I do not want it to produce a real eligibility result from bad data.

---

## 4. Unknown Values Are `"TODO"`, Not `0`

One of the smallest decisions in the project turned out to be one of the most useful.

If I have not researched a value yet, I write:

```toml
annual_value_inr = "TODO"
```

I do **not** write:

```toml
annual_value_inr = 0
```

Why?

Because `0` looks like real data.

It passes type checks, gets added into totals, and may appear to the user as:

```text
₹0
```

There is no way to tell whether that means:

> “This scheme pays nothing.”

or:

> “Nobody researched this yet.”

`"TODO"` cannot be mistaken for valid financial data.

The helper also checks inside lists:

```python
def _is_stub(v: object) -> bool:
    if v == STUB:
        return True

    if isinstance(v, (list, tuple)):
        return any(_is_stub(x) for x in v)

    return False
```

If `"TODO"` appears anywhere in the rule, the engine can return UNKNOWN instead of pretending the information is complete.

That taught me a useful rule:

> **If an unfinished state is dangerous, make it difficult to represent as valid data.**

---

## 5. A Scheme Must Pass Two Checks

Researching a scheme and verifying a scheme are not the same thing.

A scheme only becomes usable when two conditions are true:

1. every required value has been researched
2. a named human has checked those values against the official source

The code keeps those states separate.

```python
@property
def is_researched(self) -> bool:
    return not self.stubs


@property
def is_human_verified(self) -> bool:
    by = self.verified_by.strip()

    return (
        bool(by)
        and by != STUB
        and PENDING_MARKER not in by
    )


@property
def is_servable(self) -> bool:
    return (
        self.is_researched
        and self.is_human_verified
    )
```

The rest of the application only needs to check:

```python
scheme.is_servable
```

That keeps the safety rule in one place.

I learned this after a scheme once had every value filled in but had not actually been signed off by a human reviewer.

The data looked complete.

It was not verified.

Those are not the same thing.

---

## 6. Evaluation Order Matters

The rule engine follows a strict order.

```python
def evaluate(profile: Profile, scheme: Scheme) -> Result:
    """
    Order:

    1. scheme not servable
       -> UNKNOWN

    2. exclusion definitely applies
       -> INELIGIBLE

    3. criterion definitely fails
       -> INELIGIBLE

    4. something cannot be decided
       -> UNKNOWN

    5. everything passed
       -> ELIGIBLE
    """
```

The first rule matters most.

An unverified scheme cannot return a real answer in either direction.

Not even INELIGIBLE.

Because a wrong **NO** can also hurt someone by hiding a benefit they really qualify for.

And when a scheme is UNKNOWN, the financial value is also removed from the result.

```python
annual_value_inr = 0
```

That prevents an unverified rupee amount from reaching dashboards or impact numbers.

---

## 7. Rules Are Data, Not Python Code

Government scheme rules are stored in TOML files.

A simplified rule looks like this:

```toml
[[criteria]]

field = "is_unorganised_worker"
op = "eq"
value = true

ask_hi = "क्या आप असंगठित क्षेत्र में कमाई वाला काम करते हैं?"

pass_hi = "आपने बताया कि आप असंगठित क्षेत्र में काम करते हैं।"

fail_hi = "इस योजना के लिए असंगठित क्षेत्र में काम करना ज़रूरी है।"

ask_en = "Do you do income-earning work in the unorganised sector?"

pass_en = "You reported that you work in the unorganised sector."

fail_en = "This scheme requires work in the unorganised sector."

source_url = "https://www.pib.gov.in/..."
```

I chose this format for a few reasons.

Someone who understands welfare policy but does not know Python can still review the rule.

The source URL sits beside the value it supports.

And the engine never needs to know the scheme name.

It only sees:

```text
field
operator
value
```

That means adding another government scheme is mainly a research task.

It should not require rewriting the engine.

---

## 8. No Third-Party Dependencies in the Core

The project currently uses Python 3.11+ and mostly the standard library.

The main pieces are:

```text
tomllib
sqlite3
urllib
uuid
hmac
hashlib
http.server
```

This was not about trying to make the code look clever.

There were practical reasons.

### Supply chain

This project deals with information about people's benefits.

Every dependency is another project and another release process to trust.

### Deployment

The production update is close to:

```bash
git pull
systemctl restart yojana-sathi
```

There is less dependency drift to worry about.

### Auditability

If someone wants to check whether an AI model can reach the eligibility logic, they only need to inspect this codebase.

The rule I used was simple:

> Add a dependency only when the standard library cannot reasonably do the job.

So far, that has worked well.

---

## 9. Green Tests Can Still Lie

A passing test does not always mean the important code actually ran.

You can write a test for something and accidentally never reach the state you wanted to test.

So some tests also keep counters.

```python
# ! A passing test is useless if the session never reached the end.
assert ended, (
    f"[{code}] no path ever reached the end of a session"
)

# ! Make sure all expected conversation states were actually visited.
assert reached == set(State) - {...}
```

The current test run explores thousands of paths through the conversation system.

One thing surprised me.

The important bugs I found were usually **not** inside the rule engine.

They appeared around it:

* conversation state
* adapters
* dashboard
* reporting
* web proxy

The rule engine stayed stable because it is small, pure, and does almost no I/O.

That gave me another simple lesson:

> **Keep the code making important decisions as small as possible.**

---

# Four Bugs That Taught Me More Than the Features

The project still had plenty of mistakes.

These four taught me the most.

---

## Bug 1: I Counted the Same Pension Twice

Two Uttarakhand pension schemes are alternative routes to the same payment.

The scheme files know this:

```toml
exclusive_group = "uk_state_pension"
```

The rule engine correctly collapses the group before calculating the total.

So a worker sees:

```text
₹18,000
```

Correct.

But my dashboard was doing this:

```sql
SELECT
    scheme_code,
    SUM(value_inr)

FROM events

WHERE event_type = 'scheme_newly_surfaced'

GROUP BY scheme_code
```

The dashboard knew nothing about exclusive groups.

So one widow could appear as:

```text
₹36,000 entitlement surfaced
```

when the real amount was:

```text
₹18,000
```

The user saw the truth.

My statistic was wrong.

That is a dangerous kind of bug because nobody notices it by using the product.

It only shows up later in a dashboard or presentation.

The fix also needed two cases in the tests:

```python
assert split["payout"] == 54000
# One widow:
# alternative pensions collapse.


assert split["payout"] == 72000
# Two different widows:
# two real entitlements.
```

Collapsing alternatives is not the same thing as deduplicating people.

---

## Bug 2: Privacy Worked in Python but Failed in Caddy

The bot can create a temporary sheet for the worker.

The link contains a random token:

```python
secrets.token_urlsafe(16)
```

It expires after one hour and is never permanently stored.

The Python server also disables its normal access log.

```python
def log_message(self, *args) -> None:
    # ! Do not log the result-sheet URL.
    # ! The path contains a live bearer token.
    # ! Pairing that token with a client IP would create
    # ! exactly the kind of per-person tracking we avoid.
    pass
```

That looked good.

Then I put Caddy in front of the application for HTTPS.

My original configuration contained:

```text
yojanasathi.avinashnegi.com {
    log

    handle /p/* {
        reverse_proxy 127.0.0.1:8081
    }
}
```

Caddy received the request first.

Its log contained both:

```text
remote_ip
```

and:

```text
/p/<live-token>
```

So the Python process was protecting the data while the proxy was logging it.

The fix had to happen at the proxy level.

```text
format filter {
    fields {
        request>uri regexp "/p/[^\s?#]+" "/p/REDACTED"
        request>remote_ip delete
    }
}
```

That changed how I think about privacy.

> **A privacy promise inside one program is not enough. The whole system has to follow it.**

---

## Bug 3: My Comments Became False

One scheme file had a comment saying:

```text
this is why this file stays unsigned
```

That was true when I wrote it.

The next day I verified the source and signed the file.

I forgot to update the comment.

So the same file now effectively said:

```text
this file should remain unsigned
```

and:

```text
verified_by = "..."
```

Both statements had been correct at different times.

Together they made no sense.

That reminded me of something simple:

> **Comments describing what the code does can become wrong very quickly.**

Now I try to write comments explaining **why** a decision exists instead of repeating what the code already says.

---

## Bug 4: Graceful Failure Hid a Real Bug

The landing page fetches live statistics from:

```text
/stats.json
```

If the request fails, the stats section disappears.

That was intentional.

I did not want visitors seeing a broken widget.

Then I changed the site's domain.

CORS blocked the request.

The statistics disappeared exactly as designed.

So the error handling worked perfectly.

For the wrong reason.

A visible error would have told me immediately that something was broken.

Graceful degradation made the problem invisible.

The CORS helper now checks allowed origins.

```python
def allowed_origin(asked: str) -> str:
    """Return an allowed origin."""

    extra = [
        origin.strip()
        for origin in os.environ.get(
            "STATS_ORIGIN",
            "",
        ).split(",")
        if origin.strip()
    ]

    allowed = tuple(extra) + STATS_ORIGINS

    return (
        asked
        if asked in allowed
        else allowed[0]
    )
```

The response also sends:

```text
Vary: Origin
```

so a cache does not reuse the response for the wrong site.

This bug taught me that graceful failure is often good for users and bad for developers.

---

# Privacy by Not Collecting Data

One of the easiest ways to protect sensitive information is to never collect it.

The worker profile has no field for:

```text
name
phone number
Aadhaar number
```

Those values are not collected and deleted later.

They are simply not part of the profile model.

The event log only keeps broad categories such as:

```text
state
age band
occupation
income band
```

Each conversation receives a new random ID:

```python
uuid.uuid4()
```

That ID is not based on the person's Telegram or WhatsApp identifier.

So two conversations from the same person cannot automatically be linked.

---

## Counting Reach Without Storing Identity

There is one place where I need a stable identifier.

I need to avoid counting the same messaging account repeatedly when measuring reach.

For that, the messaging identifier is HMAC-hashed before being stored.

The database uses:

```sql
INSERT OR IGNORE
```

with the hash as the primary key.

But even here, I have to be careful with wording.

The dashboard says:

> **messaging accounts reached, counted once each**

It does **not** say:

> people reached

One person may use Telegram and WhatsApp.

Those appear as two different accounts because the system deliberately does not try to connect them.

The privacy design makes the metric less impressive.

But it makes it more honest.

---

# Where the Project Stands

Yojana Sathi is live.

It currently supports **seven government schemes**.

Each rule has been checked against its official source and carries a verifier name and date.

The production service runs behind Caddy and `systemd`.

But there is one number I do not want to hide:

> **Five screenings have been completed on the live bot. All five were mine.**

This is still maintainer testing.

It is not a field pilot yet.

The next problems are mostly not programming problems.

I need:

* native Hindi speakers to read the conversation naturally
* real workers to try the bot
* feedback from people who actually use government schemes
* calls to local offices when official web pages are unclear

That is probably a good sign.

The project is slowly moving from:

> **software I built**

to:

> **software that has to work for people who are not me**

---

# The Main Lesson

Before writing the first feature, I think it is worth finishing one sentence:

> **What wrong output costs my user the most, and what does that mistake cost them?**

For Yojana Sathi, mine was:

> **A wrong yes can cost someone a day's wage and a wasted journey.**

Most of the architecture followed from that.

Three-valued logic.

`"TODO"` instead of fake data.

Human verification.

AI kept outside the eligibility decision.

Temporary links.

Minimal data collection.

None of these ideas are especially clever.

They are just consequences of taking the cost of a wrong answer seriously.

---

*Yojana Sathi is open source under Apache-2.0.*

**Code:** [github.com/avinashnegi1999/yojana-sathi](https://github.com/avinashnegi1999/yojana-sathi)

**Website:** [avinashnegi.com/yojana-sathi](https://avinashnegi.com/yojana-sathi/)

**Telegram bot:** [@YojanaSathiBot](https://t.me/YojanaSathiBot)

The repository also contains `docs/BUILD_LOG.md`, where I kept the longer version with the bugs, mistakes, and wrong turns instead of cleaning them up afterwards.
