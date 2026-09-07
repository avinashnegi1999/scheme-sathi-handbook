# Yojana Sathi — 10 minute YouTube script (English)

**Language:** English — plain, simple words. No jargon walls.
**Format:** Voiceover + screen recording. No camera.
**Length:** ~1,500 words ≈ 10 minutes.

This is the same script as the Hinglish version, same beats and same timings.
Only the language changes. Screen cues are identical, so **one recording session
covers both videos** — record the b-roll once, lay two voice tracks over it.

`[square brackets]` = screen cue. Do not read them.
`⚡` = pattern interrupt — change the visual here, or the viewer drifts.

---

## 0:00 — HOOK (36 words · 15s at normal pace)

> Read this at pace, not slowly. 20–40% of viewers leave inside the first ten
> seconds, so this is the one place where speed matters more than gravitas.

[screen: black. White text: ₹455]

One day of work. Four hundred and fifty-five rupees.

[screen: text changes to "1 wrong paper"]

A man gives up that whole day, pays the bus fare, walks to a government office —
and gets sent home because he brought the wrong paper.

[screen: text — "So I built an app."]

So I built an app.

⚡

---

## 0:15 — WHO I AM (135 words)

[screen: terminal, `neofetch` or your desktop. Then the GitHub profile.]

Hello. I'm Avinash Negi, and today I'll tell you the whole story of this app —
what got built, what broke, and what is still wrong with it.

I'm from Kotdwara, a small town in Uttarakhand. There is no tech industry here.
I'm doing an online degree, and I'm learning Python backend development mostly on
my own — YouTube, documentation, and a lot of trial and error.

And let me be honest with you up front: I don't have a job yet. No company
experience. Just projects.

So if that's you too — degree in progress, no job, and no idea what to build that
actually matters — this video is for you.

Because I'm not showing you a tutorial. I'm showing you one real project, all the
way through. The good parts and the bad ones.

⚡

---

## 1:00 — THE PROBLEM (150 words)

[screen: myScheme.gov.in scrolling on laptop. English text, filters, forms.]

So, the problem.

India has a lot of government welfare schemes. Insurance, pensions,
registrations. They exist. The money is already allocated.

The problem is information.

All of it is online — but in English. You need to be able to read, you need a
browser, and you need to know what "land holding in hectares" means.

Now picture a mason. A driver. A daily wage labourer. The people who need these
schemes the most — for them, that's three assumptions too many.

So what do they do? They walk there. They give up a day. And if the answer turns
out to be no, or they brought the wrong document, that's four hundred and
fifty-five rupees gone, plus the fare.

That's the real cost. And this whole project is built around that one number.

⚡

---

## 2:00 — THE PROJECT I THREW AWAY (115 words)

[screen: the Saans repo on GitHub. Scroll it. Then close the tab.]

Now something I could have hidden, but won't.

This is my second project. The first one was called Saans — a personal air
pollution agent on Telegram. Around three and a half thousand lines. Zero
dependencies. All checks passing. A live bot. A public repo.

That is finished code. And I parked it on the thirtieth of August.

Why? Because I never deployed it. I never put it in front of a single real user.

And at first I told myself a different reason — that the idea already existed
somewhere else. But honestly, that was a feeling. Having zero users is not a
feeling. That one is a score.

The lesson: deploy first. Polish later.

⚡

---

## 2:45 — FOUR DECISIONS BEFORE ANY CODE (190 words)

[screen: `sathi/rules/engine.py` open. Then `tests/test_rules.py`.]

Before writing any code for this one, I made four decisions. And I never went
back on them.

**One. The rule engine is not allowed to call an AI model.**

The model can handle conversation. It can rephrase Hindi. But it never sees a
threshold, it never produces a rupee figure, and it never decides who qualifies.
Eligibility is decided by ordinary, boring, readable Python. And there's a test
that fails the build if anyone ever imports a model in there by accident.

**Two. Three answers, not two.**

Every rule returns yes, no, or **I don't know**. Missing information is never
quietly treated as a no.

Think about why. A wrong yes means someone loses a day's wages for nothing. A
wrong no means they never get a scheme they were entitled to. But "I don't know —
go ask this exact question at the centre" costs nothing, and it's still useful.

**Three. Any value I haven't researched is literally the text "TODO". Even
numbers.**

Never zero. Because a zero looks like somebody checked.

**Four. There is no name, phone number, or Aadhaar field anywhere.**

Not "we don't store it" — the field does not exist. So it cannot be stored by
accident.

⚡

---

## 4:00 — HOW IT ACTUALLY WORKS (150 words)

[screen: phone recording. WhatsApp. /start → language → consent → questions.]

So here's how it actually works.

It's a conversation. On WhatsApp and on Telegram. In Hindi or English.

It asks what a clerk would ask — which state you're in, your age, what work you
do, roughly what you earn.

[screen: tapping buttons, then the state question rendering as a list]

Every answer is a button. Nothing to type. Nothing to spell.

[screen: results screen, scroll slowly through the ₹ figures]

And at the end it tells you three things. What you qualify for. What it's worth
per year, in rupees. And where to walk to claim it.

Two lakh rupees of accident cover, for twenty rupees a year. Three thousand
rupees a month as a pension from the age of sixty.

And e-Shram — which pays nothing on its own. So the app says it pays nothing,
instead of inventing a number to look more impressive.

⚡

---

## 5:00 — RE-HOOK (50 words)

[screen: cut to black. Then red text: "Now the mistakes."]

So far I've shown you what went right.

Now I'll show you what I got wrong. Because if you only see the good half, you'll
think this went smoothly. It did not.

I found six bugs in the first hour.

---

## 5:20 — WHAT BROKE (185 words)

[screen: BUILD_LOG.md, bugs section, scrolling]

The bot went live. I used it myself, like a worker would. Six bugs in one
session.

And notice this — **every single one was in the conversation layer. Not one was
in the rule engine.**

For example: sending an empty keyboard made Telegram return a bare four-hundred
error. Nothing crashed. Nothing logged. The session just died silently.

There was a loop with no way out. I hit it on my first run by typing "i dont do
any job".

[screen: the `is_verified` property in the diff]

But the worst bug, I didn't find at all. I made the repo public and had an AI
review it. It caught something genuinely serious.

In my code, "verified" was implemented as "there are no TODOs left in this file".
But all three scheme files were already fully researched — so none of them had a
TODO. Which meant the code was reporting them as verified, when no human being
had actually confirmed a single number.

One property. One wrong line. And the entire safety gate was doing nothing.

⚡

---

## 6:30 — CTA (50 words)

[screen: the GitHub repo, then a subscribe animation]

If this kind of thing is interesting to you — the whole thing is open source,
link's in the description. The build log is there too, with every mistake in it.

And if you're trying to build your first real project, subscribe. This is what I
do here, publicly.

---

## 6:50 — SHIPPING IT (180 words)

[screen: terminal, ssh into the EC2 box. `systemctl status sathi`.]

Now, deployment.

This runs on a small AWS server in the Mumbai region. About ten dollars a month.

[screen: `deploy/install-on-vm.sh` scrolling]

Deployment is one script. It copies the code, keeps the secrets separate, and
restarts the service. And — this part matters — **it runs the full test suite
before deploying.** If a test fails, the deploy doesn't happen.

It runs under systemd, which means if the server reboots, it comes back on its
own.

[screen: Telegram bot conversation]

Telegram went live first. Telegram is easy — get a token, start polling, done. No
verification, no business account.

And I did Telegram first on purpose. Because if I had waited for WhatsApp,
Meta's approval process could have blocked my deployment date. So I wrote the
core to be channel-agnostic — adding a channel should be one file, not a rewrite.

That decision paid off later.

⚡

---

## 8:00 — WHATSAPP, AND TWO BUGS (195 words)

[screen: Meta developer dashboard, WhatsApp configuration page]

Now WhatsApp.

The WhatsApp Cloud API is a lot more complicated than Telegram. Create an app,
create a business portfolio, get a phone number ID, get an app secret, set up a
webhook, and it has to be HTTPS — plain HTTP won't work.

I set it all up. The dashboard said everything was fine. Green tick.

And then — nothing. Not a single message reached my webhook.

[screen: the `subscribed_apps` API response]

It took two hours. The reason: that "messages — Subscribed" toggle I switched on
in the dashboard had not subscribed my app at all. It had subscribed Meta's own
demo app.

One API call to find it. One API call to fix it. The dashboard was lying.

[screen: the sqlite error in the journal]

The second bug was better. WhatsApp started working — but every message got the
same reply: "something went wrong, please send /start again."

The reason: my database connection was created on one thread, and WhatsApp was
writing from another. SQLite does not allow that.

Telegram never showed this bug, because Telegram runs on a single thread. **Old
code, and the bug only appeared when a second channel arrived.**

⚡

---

## 9:15 — WHAT'S STILL WRONG, AND CLOSE (140 words)

[screen: `docs/BUILD_LOG.md` — "What is still open, and what is still wrong"]

One last thing. And this is the part people leave out of videos.

This project is not finished.

No human has signed off on any of the three schemes yet. Which means that today,
the app tells every worker "I don't know" — for every scheme. On purpose. Because
until someone puts their name against a number, the engine refuses to give a
verdict.

And there's an open question about e-Shram's age limit. One official page says
sixteen and above. Another says sixteen to fifty-nine. If the second one is
correct, then every worker over sixty is getting a wrong answer right now.

I could have hidden that. But this entire project exists for one idea — when you
don't know, say you don't know.

[screen: end card]

I'm Avinash Negi. Code's in the description. See you in the next one.

---

# PRODUCTION NOTES

## Word count by segment

| Segment | Words | Running |
|---|---|---|
| Hook | 36 | 0:15 |
| Who I am | 135 | 1:00 |
| Problem | 150 | 2:00 |
| Parked project | 115 | 2:45 |
| Four decisions | 190 | 4:00 |
| How it works | 150 | 5:00 |
| Re-hook | 50 | 5:20 |
| What broke | 185 | 6:30 |
| CTA | 50 | 6:50 |
| Shipping | 180 | 8:00 |
| WhatsApp bugs | 195 | 9:15 |
| Close | 140 | 10:00 |
| **Total** | **~1,575** | |

If it runs long, cut from **Four decisions** or **How it works**.
Never cut the Hook, the Re-hook at 5:00, or the Close.

## Recording both videos from one session

The screen cues in this file and in the Hinglish file are **identical and in the
same order**. So:

1. Record the b-roll once, following the shot list.
2. Record two voice tracks — one Hinglish, one English.
3. Lay each over the same footage.

You get two videos for slightly more than the effort of one. Only the voice
track and the burned-in titles change.

## Shot list

Identical to the Hinglish script — see `17-youtube-script-10min-hinglish.md`.

**Blocking:** the results-screen shot needs the schemes signed off, otherwise the
app answers "I don't know" to everything and there is nothing to film. Keep
**one** scheme unsigned on purpose so you also capture a real "I don't know" for
the closing section.

## Title options

1. I built an app that says "I don't know" — and that's the whole point
2. 6 bugs in the first hour: building a real product with no job and no team
3. From a small town in Uttarakhand to AWS — one project, start to finish

## Description skeleton

```
I built a WhatsApp and Telegram bot that tells India's unorganised workers which
government schemes they qualify for — in Hindi, using buttons, with no typing.

This video is the whole story: why I built it, how I built it, what broke, and
what is still wrong with it.

Code (open source): github.com/avinashnegi1999/yojana-sathi
Build log (every mistake): github.com/avinashnegi1999/scheme-sathi-handbook
Telegram: @YojanaSathiBot

00:00 ₹455
00:15 Who I am
01:00 The problem
02:00 The project I threw away
02:45 Four decisions
04:00 How it works
05:20 What broke
06:50 Deploying to AWS
08:00 Two WhatsApp bugs
09:15 What's still wrong
```

**Tags:** python, backend, aws ec2, whatsapp cloud api, telegram bot, build in
public, indian developer, self taught developer, side project

---

# NUMBERS THIS SCRIPT USES — all verified 2026-09-07

₹455/day male and ₹315/day female casual labourer · 3 schemes · PMSBY ₹2,00,000
cover for ₹20/year · PM-SYM ₹3,000/month from age 60 · e-Shram ₹0, a gateway ·
zero third-party dependencies · 2 channels · AWS ap-south-1 · ~$10/month ·
Saans ≈ 3,500 lines · 6 bugs in the first hour.

## Do NOT add to this script

- **Any count of unorganised workers in India.** Not sourced anywhere in the
  repo. AI summaries of this project have invented "44M+" before.
- **"Zero persistence"** — false, there is a database with two tables.
- **"No hallucinations" / "0% hallucination"** — the model is optional and off.
  That is a design choice, not a proof.
- **"Works without a smartphone"** — both channels need one.
- **PM-KISAN, Ayushman Bharat** — not in this project.

Quoting an unverifiable number is the exact failure this project exists to
prevent. A public video is the worst possible place to slip.
