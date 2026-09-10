# I built a welfare bot that refuses to answer

### The most useful thing it says is "I don't know"

---

A man in Uttarakhand takes a day off work to visit a government service centre. He has heard there is a pension for people his age. He borrows a seat in a shared taxi, waits in a queue, reaches the counter — and is told he does not qualify. Or that he is missing a certificate nobody mentioned.

He has lost a day's wage. For a male casual labourer in India that averaged **₹455** in 2025. For a woman, **₹315**.

Most people do not come back for a second try.

That is the problem I spent six weeks on. Not "people don't know about government schemes" — often they do. The harder question is the one you cannot answer from outside the building: *will this scheme actually accept me?*

---

## The thing that already exists

India has a portal for this. It is called myScheme, it is well built, and it is a website in English with a form.

Now picture who we just described. A construction worker. A widow in a hill district. Someone whose phone is a smartphone but whose reading is not fluent — in a language the form does not offer.

So I did not try to replace myScheme. I tried to do the last mile on top of the same government data: **a conversation in spoken Hindi that ends with a filled application sheet and an address to walk to.**

You tap a few buttons. It tells you which schemes you qualify for, what each is worth in rupees, which papers to carry, and where to go.

---

## The decision everything rests on

People assume the interesting part is the AI. It isn't. The interesting part is where the AI is not allowed to go.

**A language model never decides whether you qualify.**

Not "we check its work." Not "we prompt it carefully." It is structurally absent — the code that decides eligibility cannot even import it, and a test fails if anyone tries.

Here is why I was that strict. When a model invents a threshold — says the income limit is ₹15,000 when it is ₹10,000 — nobody finds out at a keyboard. Somebody finds out at a counter, sixty kilometres away, having spent a day's pay to get there.

A confident wrong answer costs more than no answer.

So the rules live in plain text files, each value carrying a link to the government page it came from, and a human reads every one before the bot is allowed to say it out loud.

---

## The refusal

Most software has two answers: yes and no.

This has three. **Eligible. Not eligible. And I don't know.**

The third appears whenever something is genuinely unknown — you skipped a question, or the rule itself hasn't been checked against an official source yet. It never guesses. It never fills a gap with a sensible-looking default.

That sounds like a weakness. It's the opposite. The refusal is what makes the answers worth anything.

One more line travels with every rupee figure — in the chat, on the printed sheet, everywhere:

> **यह पैसा अभी मिला नहीं है।**
> *This money has not arrived. To get it, you have to apply.*

An eligibility screening is not an approval. If a worker walks away believing the government has promised her ₹18,000, I've done harm, not good.

---

## What I got wrong

This is the part worth reading.

**I killed the project before this one.** Better looking, weeks of work, never deployed, never put in front of a single person. A score sheet taught me the lesson cheaply: *building feels like progress; only shipping is measured.* Then I repeated the mistake here anyway — parked the deploy for days behind an imagined "clear day". The actual deploy took under an hour.

**My impact number lied to me.** Two Uttarakhand pensions are alternative routes to the same payment; you get one, not both. The bot told workers this correctly from day one. My own dashboard added them together, so a single widow showed up as ₹36,000 of "entitlement surfaced" against a real ₹18,000.

Notice the direction. The *user* got the truth. The *statistic about* the user was inflated. That's the worse way round, because you never catch an inflated number by using the product. It just quietly makes its way into a slide.

**I built a privacy feature and then leaked it.** The result sheet is served as a link with a random one-hour token. I was careful: nothing written to disk, and the server deliberately refuses to log which sheet was opened, because that log would pair someone's IP address with a live link to their benefits.

Then I put a standard web server in front of it for encryption. That server saw every request first — and logged precisely what I'd taken such trouble to avoid.

A privacy promise kept inside one program is not a promise. Every part of the system has to agree.

**And one file argued with itself.** The weakest-sourced scheme carried a comment I'd written honestly: *"this is why this file stays unsigned."* I signed it the next day and never updated the comment. For a day, that file told every reader its own signature shouldn't exist.

Both sentences were true when written. Nothing forced them to agree. Comments describing a decision rot the moment the decision changes, and no test reads prose.

---

## Where it honestly stands

The bot is live. Seven schemes, each read line by line against its official source, each carrying my name and the date I checked it. It answers in Hindi or English and hands you a printable sheet at the end.

**Five people have completed a screening. All five are me.**

The landing page prints that number, pulled live from the bot, captioned *still maintainer testing, not a field pilot*. It shows nothing at all until the count rises above zero — a counter reading `0` claims less than the honest sentence sitting beneath it.

An outside review rated the engineering 8.9 out of 10, and the same project 65 out of 100 against the competition's rubric. Both are fair. A quarter of that rubric is proven impact, and mine is worth about three of those twenty-five points.

The distance between those numbers is not a code problem. Every remaining blocker is a person: a Hindi speaker who hasn't yet read the script aloud, five workers not yet recruited, a phone call to a district office that a fourth reading of a web page will not settle.

For something built alone, that's a good place to be stuck. I'd rather say it plainly than dress it up.

---

## The one idea worth stealing

Before writing any code, finish this sentence: **the wrong output that costs my user the most is ___, and it costs them ___.**

Mine was: *a wrong yes costs a day's wage and a wasted journey, and most people don't come back.*

Everything else was downstream of that. Three-valued answers. A citation on every number. A model kept out of the decision. A bot whose most important response is that it doesn't know.

You don't need a big project to try this. You need to be honest about what your software's worst day looks like for someone who isn't you.

---

*Yojana Sathi is open source, Apache-2.0.*
*Code: [github.com/avinashnegi1999/yojana-sathi](https://github.com/avinashnegi1999/yojana-sathi) · Site: [avinashnegi.com/yojana-sathi](https://avinashnegi.com/yojana-sathi/) · Try it: [@YojanaSathiBot](https://t.me/YojanaSathiBot)*

*A much longer and far more technical version — every bug, every wrong turn, nothing tidied up — lives in the repository. This is the short one.*
