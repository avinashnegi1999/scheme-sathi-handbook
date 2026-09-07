# Yojana Sathi — full narration script

**Read this out loud, or feed it to TTS, exactly as written.**
Word count ≈ 520. At a calm pace that is **about 3 minutes**.

`[square brackets]` are visual cues — do not read them.

---

[Visual: you on camera, or a title card with the project name]

Hello. I am Avinash Negi, and I built Yojana Sathi.

[Visual: title card — ₹455]

Let me start with a number. A male casual labourer in India earns about four
hundred and fifty-five rupees on a day he works. A woman earns about three
hundred and fifteen.

Now imagine that person hears there is a government scheme he might qualify for.
He takes a day off. He pays the bus fare. He walks to the office. And he is told
he brought the wrong paper, or that he was never eligible in the first place.

That is not an inconvenience. That is a day's wage and the fare, gone, to find
out something a two-minute conversation could have told him.

[Visual: myScheme.gov.in on a laptop, scrolling]

The information does exist. It is online. But it assumes you can read English,
that you have a browser, and that you know what "land holding in hectares"
means. For the worker who most needs these schemes, that is three assumptions too
many.

[Visual: phone screen recording — WhatsApp, /start, language buttons]

So I built a conversation instead.

Yojana Sathi runs on WhatsApp and on Telegram, in Hindi or in English. It asks
the same questions a clerk would ask — your state, your age, the work you do,
roughly what you earn.

[Visual: tapping through consent, then two or three questions]

Every answer is a button. Nothing has to be typed and nothing has to be spelled.

And it never asks your name, your phone number, or your Aadhaar number. Those
fields do not exist anywhere in the code — so they cannot be stored by accident,
even if I wanted them.

[Visual: the state question, which renders as a scrollable list]

[Visual: results screen — scroll slowly through the ₹ figures]

At the end, it tells you three things. What you qualify for. What it is worth, in
rupees, per year. And where to walk to claim it.

Two lakh rupees of accident cover, for twenty rupees a year. Three thousand
rupees a month as a pension, from the age of sixty. And an e-Shram number — which
pays nothing by itself. So the app says it pays nothing, instead of inventing a
figure to look more impressive.

[Visual: an UNKNOWN verdict on screen]

But the part I care most about is this one.

There is a third answer. Not yes, and not no — "I don't know".

When the rules for a scheme have not been confirmed by a human being, this app
does not guess. It says it does not know, and then it gives the worker the exact
question to ask at the centre.

[Visual: a scheme .toml file open — verified_by and source_url visible]

Every number in this system carries a link to the government page it came from,
and the name of the person who checked it. Until somebody puts their name against
a scheme, the engine refuses to give a verdict on it at all.

A wrong yes costs a worker a day's wage. Silence costs nothing. So when I am not
sure, I stay silent.

[Visual: code — sathi/rules/ directory]

One more decision. The eligibility engine cannot call a language model. Not "does
not" — cannot. The model handles conversation and translation. The rules decide
entitlement, and they are ordinary, readable, testable code.

[Visual: split screen — Telegram and WhatsApp answering the same question]

One rule engine. Two channels. No third-party dependencies — the Python standard
library and nothing else. Running right now on a small server in Mumbai.

[Visual: end card with links]

It is open source, and it is honest about what it does not yet know.

Thank you for watching. This is Yojana Sathi.

---

## End card text

```
Yojana Sathi  ·  योजना साथी
github.com/avinashnegi1999/yojana-sathi
Telegram: @YojanaSathiBot
Avinash Negi
```

---

## Before you record

1. **Sign off the scheme files first.** Right now every scheme says
   `PENDING HUMAN VERIFICATION`, so the app answers UNKNOWN to everything and the
   results section of this video cannot be filmed.
2. **Keep one scheme unsigned on purpose** while filming. Then you get real ₹
   verdicts and a genuine UNKNOWN in the same recording. Sign it off afterwards.
3. Practise once out loud. The two places to slow down are the ₹455 line at the
   start and the "I don't know" section — those are the two ideas a judge should
   still remember tomorrow.

## Numbers in this script — all verified 2026-09-07

₹455/day male and ₹315/day female casual labourer · 3 schemes · PMSBY ₹2,00,000
cover for ₹20/year · PM-SYM ₹3,000/month from age 60 · e-Shram ₹0, a gateway ·
zero third-party dependencies · 2 channels · AWS ap-south-1 (Mumbai).

**Do not add any number that is not in this list.** In particular, do not state
how many unorganised workers there are in India — that figure is not sourced
anywhere in the repo, and it has been invented by AI summaries of this project
before. Quoting an unverifiable number in the video is the exact failure the
project exists to prevent.
