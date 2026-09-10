# A rule engine that isn't allowed to guess

### Building a welfare-eligibility bot where the expensive failure is a confident wrong answer

---

Most software fails by crashing. This one fails by being confidently wrong, and the cost lands on someone else.

The user is an unorganised-sector worker in India — construction, farm labour, domestic work. The bot screens them against government welfare schemes and tells them what they qualify for, what it's worth, and where to go. If it says *yes* and the answer is *no*, that person takes a day off work, borrows a seat in a shared taxi, queues, and is turned away at the counter. A male casual labourer's daily earnings averaged **₹455** in 2025; a woman's, **₹315**.

So the design question wasn't "what features?" It was: **what is the wrong output that costs the user most, and in what unit?**

Everything below is downstream of that one sentence.

---

## Constraint 1: no language model in the decision path

The rules live in `sathi/rules/`. That package may not import a model, an HTTP client, or a socket. Not by convention — enforced:

```python
def test_engine_does_not_import_an_llm():
    code = (
        "import sys; import sathi.rules.engine;"
        "banned={'urllib.request','http.client','socket','ssl',"
        "'anthropic','openai','requests'};"
        "bad=[m for m in sys.modules if m in banned or 'llm' in m.lower()];"
        "print(','.join(sorted(bad)))"
    )
    out = subprocess.run(
        [sys.executable, "-c", code], cwd=ROOT,
        capture_output=True, text=True, check=True
    ).stdout.strip()
    assert out == "", f"rules/ pulled in {out}"
```

It runs in a **subprocess**, because `sys.modules` in the test process is already polluted by the test runner itself. And it checks module names exactly rather than by prefix — `urllib.parse` is pure string handling and shows up innocently via `pathlib`, so a prefix match would either false-positive or force me to loosen it until it meant nothing.

The model still has a job: mapping free text to a category, and rephrasing Hindi a human wrote. It never sees a threshold, never produces a ₹ figure, never produces a verdict. And every free-text mapping is confirmed back to the user before it's recorded.

The whole flow also works with `LLM_API_KEY` unset. That's a tested configuration, not a degraded one — buttons and templated Hindi produce identical verdicts.

---

## Constraint 2: three-valued logic, all the way down

Booleans are the wrong type for this problem. A missing answer is not `False`.

```python
def apply(op: str, actual: object, expected: object) -> bool | None:
    """Compare one profile value against one scheme threshold.

    Returns None when the answer is genuinely unknown. Raises OperatorError
    when the comparison is nonsense (a scheme-file bug the caller reports,
    never hides).
    """
    if op == "exists":
        # * The one operator decidable with no answer: absence IS the answer.
        return actual is not None

    if _is_stub(expected):
        return None
```

Three outcomes, deliberately distinct:

| Return | Meaning |
|---|---|
| `True` / `False` | decided |
| `None` | genuinely unknown — missing answer, or an unresearched rule |
| `OperatorError` | the scheme file is broken; surface it, never swallow it |

That last row matters. Comparing a boolean against an age band is a *data* bug. If it silently evaluated — `bool` is a subclass of `int` in Python, so `True` quietly becomes `1` — a malformed scheme file would produce real verdicts:

```python
def _num(v: object, where: str) -> int | float:
    if isinstance(v, bool) or not isinstance(v, (int, float)):
        raise OperatorError(f"{where}: expected a number, got {v!r}")
```

Stub propagation is recursive, because a threshold can be a list:

```python
def _is_stub(v: object) -> bool:
    if v == STUB:
        return True
    if isinstance(v, (list, tuple)):
        return any(_is_stub(x) for x in v)
    return False
```

---

## Constraint 3: an unresearched value is `"TODO"`, never `0`

This is the smallest decision with the biggest payoff.

```toml
annual_value_inr = "TODO"   # not 0
```

A zero looks researched. It passes type checks, sums cleanly into totals, renders as "₹0" — and no validator can distinguish "this scheme pays nothing" from "nobody has looked this up yet." A string in an integer field cannot be mistaken for either. It propagates to `None` through the operators and surfaces as UNKNOWN.

**Make the unfinished state un-representable.** Every time I moved a safety property from a comment into the type system, it stayed fixed. Every time I wrote it in prose, it rotted — and I have two examples later in this post of exactly that.

---

## The two gates

Filling in every value isn't the same as verifying it. One person reading a PDF once is one person reading a PDF once — and I was that person, transcribing figures at 1am.

```python
@property
def is_researched(self) -> bool:
    """No "TODO" left in the file — somebody looked every value up."""
    return not self.stubs

@property
def is_human_verified(self) -> bool:
    """A named human confirmed every value against the official source."""
    by = self.verified_by.strip()
    return bool(by) and by != STUB and PENDING_MARKER not in by

@property
def is_servable(self) -> bool:
    """May the engine return a real verdict? Both gates."""
    return self.is_researched and self.is_human_verified
```

`is_servable` is the only property anything checks. Both states routed through one place so they can't drift apart again — they had, once, and a scheme with every value filled but nobody's signature was serving real ELIGIBLE verdicts off unchecked transcription.

The engine's precedence order, with servability first:

```python
def evaluate(profile: Profile, scheme: Scheme) -> Result:
    """Decide one scheme for one worker. Pure function.

    Precedence, in order — a definite NO outranks a maybe:
      1. scheme not servable            → UNKNOWN, always, no further checking
      2. any exclusion definitely hits  → INELIGIBLE
      3. any criterion definitely fails → INELIGIBLE
      4. anything undecidable           → UNKNOWN
      5. otherwise                      → ELIGIBLE
    """
```

Rule 1 returning UNKNOWN before checking anything is the point. An unverified scheme cannot produce a verdict *in either direction* — a wrong NO is a missed entitlement, which is just as much a failure as a wrong YES.

And when it does return UNKNOWN, the ₹ value is zeroed on the way out:

```python
annual_value_inr=0,  # ! never let an unverified ₹ reach a metric
```

---

## Rules as data, with a citation per value

Scheme rules are TOML, not Python. Each criterion carries its own source URL and its own worker-facing text in both languages:

```toml
[[criteria]]
# * Source comparison 2026-09-08: worker status is a separate condition.
# * Occupation and zero income do not establish or disprove this by themselves.
field      = "is_unorganised_worker"
op         = "eq"
value      = true
ask_hi     = "क्या आप असंगठित क्षेत्र में कमाई वाला काम करते हैं?"
pass_hi    = "आपने बताया कि आप असंगठित क्षेत्र में काम करते हैं।"
fail_hi    = "इस योजना के लिए असंगठित क्षेत्र में काम करना ज़रूरी है।"
ask_en     = "Do you do income-earning work in the unorganised sector?"
pass_en    = "You reported that you work in the unorganised sector."
fail_en    = "This scheme requires work in the unorganised sector."
source_url = "https://www.pib.gov.in/PressReleasePage.aspx?PRID=2293891&..."
```

Three properties fall out of this that Python-coded rules wouldn't give you:

1. **Auditable by a non-programmer.** Someone who knows welfare policy but not Python can check a rule against its source.
2. **The citation lives beside the value**, not in a doc that drifts.
3. **The engine can't special-case a scheme**, because it never sees scheme names — only fields, operators and values.

Adding a scheme is research work, not engineering work. That's the whole reason for the format.

---

## Zero dependencies

Python 3.11+, standard library only. `tomllib` for the rules, `sqlite3` for the event log, `urllib` written directly against the Telegram and WhatsApp APIs, `assert` for tests.

Not asceticism. Three concrete reasons:

- **Supply chain.** This handles decisions about people's benefits. Every dependency is somebody else's release process.
- **Deployment.** `git pull && systemctl restart`. No virtualenv drift, no resolver, nothing to break on a `t4g.micro`.
- **Auditability.** A reviewer checking whether a model touches the decision has to read one codebase, not a tree.

The rule I set: adding one needs a stated reason the stdlib can't do the job. In six weeks nothing cleared it. The pack is HTML rather than PDF for exactly this reason — the stdlib has no PDF writer, and HTML prints fine from a phone.

---

## Testing: coverage counters, because green tests lie

The suite walks every button at every screen in both languages. But a passing test that never reached the code it claims to check is worse than no test — it's an alibi. So the tests assert their own coverage:

```python
# ! Counters, because a test that never reached the thing it checks is the trap
assert ended, f"[{code}] no path ever reached the end of a session"
...
assert reached == set(State) - {...}
```

Current run: **2,541 paths, 217 completed sessions.** Three bugs got past this suite in one day before the counters existed, which is why they exist.

The number that matters more: **not one bug I found lived in the rule engine.** They were all in the layer between the worker and the arithmetic — the adapter, the conversation state, the dashboard, the proxy. The engine survives because it's small, pure, and has no I/O. That's a decent argument for keeping the deciding core as small as you can get away with.

---

## Four bugs worth the words

### 1. The same double-count, one layer up

Two Uttarakhand pensions are alternative routes to the same payment. Scheme files declare it:

```toml
exclusive_group = "uk_state_pension"
```

and `engine.value_totals()` collapses a group to its largest member before summing. The worker sees ₹18,000. Correct.

The **dashboard** did this:

```sql
SELECT scheme_code, SUM(value_inr)
FROM events
WHERE event_type='scheme_newly_surfaced'
GROUP BY scheme_code
```

No sessions, no groups. One widow reported as ₹36,000 of "entitlement surfaced" against a real ₹18,000.

Note the direction: the *user* got the truth, the *statistic about* the user was inflated. Nobody catches an inflated statistic by using the product.

I wrote `exclusive_group` to prevent this, then wrote a second place computing the same quantity that didn't use it. **A fix living in one function protects one caller.** `templates.py` and `pack.py` were both made to call `value_totals()`; `report.py` never was, because it reads the event log rather than a live result.

The fix groups per session — and the second assertion matters as much as the first:

```python
assert split["payout"] == 54000   # one widow: pensions collapse, plus PM-SYM
assert split["payout"] == 72000   # two DIFFERENT widows: two entitlements
```

Collapsing is not deduplication.

### 2. A privacy property that stopped at the process boundary

The application sheet is served as a link with a `secrets.token_urlsafe(16)` token, held in a module-level dict, expiring in an hour, revoked by `/clear`, never written to disk. The server suppresses its own access log deliberately:

```python
def log_message(self, *args) -> None:
    # ! Silence, deliberately. The default access log records the path,
    # ! which is the token, and the client address. Logging which pack was
    # ! opened from where is exactly the per-person analytics this project
    # ! promises not to keep.
    pass
```

Then I put Caddy in front for TLS, from my own runbook:

```
yojanasathi.avinashnegi.com {
	log
	handle /p/* { reverse_proxy 127.0.0.1:8081 }
}
```

Caddy sees the request first. Its log immediately held `"remote_ip"` next to `"uri":"/p/<live token>"` — a client IP paired with a valid bearer link to someone's benefits sheet.

**A privacy property enforced inside one process is not a property of the system.** Fixed at the proxy, not by remembering:

```
format filter {
	fields {
		request>uri regexp "/p/[^\s?#]+" "/p/REDACTED"
		request>remote_ip delete
	}
}
```

### 3. Two files that contradicted themselves

`uk_widow.toml` carried a comment I wrote honestly on the 9th — *"this is why this file stays unsigned"* — four lines above a signature I added on the 10th. Both true when written; nothing forced them to agree.

For most projects that's a doc bug. Here provenance *is* the product, so a file contradicting its own provenance attacks the central claim.

Its sibling had two statements that had become outright false: it claimed a condition was "now encoded below" that had since been removed, and claimed a question asked the stricter of two readings when it asks the other one.

**Comments describing a decision rot the moment the decision changes, and no test reads prose.** The only defence I've found: write down *why*, not *what*, and re-read the header whenever the body changes.

### 4. A graceful degradation that hid a real failure

The landing page fetches `/stats.json` and hides its whole section if the request fails — so the page degrades to exactly what it was before the feature existed.

Then a custom domain moved the site's origin (GitHub applies a user-level domain to every project page). CORS blocked the fetch. The section hid itself, working perfectly, for entirely the wrong reason.

A skeleton or spinner would have screamed. Hiding was right for a worker and wrong for me.

One header can't name two origins, so the server echoes back whichever allowed origin asked:

```python
def allowed_origin(asked: str) -> str:
    """The origin to echo back, or the primary one when unknown."""
    extra = [o for o in os.environ.get("STATS_ORIGIN", "").split(",") if o.strip()]
    allowed = tuple(o.strip() for o in extra) + STATS_ORIGINS
    return asked if asked in allowed else allowed[0]
```

With `Vary: Origin`, so a cache can't replay one origin's response to another. Never `*` — the endpoint answers strangers.

---

## Privacy by absence

There is no name field. No phone field. No Aadhaar field. Not stripped — **never defined**, so they cannot be stored by accident.

The event log holds coarse bands only (state, age band, occupation, income band) under a fresh `uuid4` per conversation that is not derived from the messaging id. Two sessions by the same person are unlinkable by design.

The one place a stable identifier is needed — counting unique reach — uses an HMAC of the channel id with `INSERT OR IGNORE` on a primary key as the only dedupe guard. And the dashboard label says what the data actually supports:

> messaging accounts reached, counted once each

Not "people". One human with a Telegram account and a WhatsApp number is two rows, because the identifiers hash differently and nothing links them — the same design that stops us knowing who anyone is.

---

## Where it stands

Live on Telegram, seven schemes, each value read against its official source and signed with a name and date. Runs under `systemd` on a `t4g.micro` behind Caddy, with a Docker path that runs the full suite at image build time.

**Five screenings have completed on the live bot. All five are mine.**

The landing page prints that number, fetched live, captioned *still maintainer testing, not a field pilot* — and renders nothing at all until the count exceeds zero, because a counter reading `0` claims less than the honest sentence beneath it.

Every remaining blocker is person-shaped: a native Hindi speaker who hasn't read the script aloud, workers not yet recruited, a phone call to a district office that a fourth reading of a web page won't settle.

---

## The transferable part

**Write down the expensive mistake before the first line of code.** Not a feature list. One sentence: *what wrong output costs my user most, and in what unit?*

Mine was: *a wrong yes costs a day's wage and a wasted journey, and most people don't come back.*

Three-valued verdicts, `"TODO"` over `0`, a two-gate servability check, a model structurally barred from the decision — none of those were clever. Each is the obvious consequence of that sentence. The sentence did the work.

---

*Apache-2.0. Code: [github.com/avinashnegi1999/yojana-sathi](https://github.com/avinashnegi1999/yojana-sathi) · Site: [avinashnegi.com/yojana-sathi](https://avinashnegi.com/yojana-sathi/) · Bot: [@YojanaSathiBot](https://t.me/YojanaSathiBot)*

*`docs/BUILD_LOG.md` in the repo is the unedited version — every bug and wrong turn, nothing tidied up afterwards.*
