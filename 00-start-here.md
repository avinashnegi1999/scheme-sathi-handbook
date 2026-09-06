# Start here

## What this project is, in one paragraph

Scheme Sathi is a Telegram bot for India's unorganised workers — construction
labourers, domestic workers, street vendors, farm hands. It asks them a short
series of button-tapped questions in Hindi or English, runs their answers
through a rule engine that has no language model in it, and tells them which
government welfare schemes they qualify for, what each is worth in rupees a
year, and which office to walk into with which documents.

## Why it exists

The Government of India runs `myScheme.gov.in`, which lists every welfare
scheme and lets you filter them. It is a good site. It also assumes you can
read, that you have a browser, and that you can work out which of 3,000 schemes
applies to you.

The person who most needs those schemes has none of that.

So this is **last-mile delivery on top of a government system**, never a
replacement for it. That distinction matters and I should always say it out
loud: it shows the existing solution was checked before building a new one.

## The one idea worth carrying out of this project

**Being unsure is a first-class answer.**

Most systems are built so every code path produces a confident output. This one
has three verdicts — `ELIGIBLE`, `INELIGIBLE`, and `UNKNOWN` — and `UNKNOWN` is
not a failure state. It is what the system returns when it has not verified
something, and it comes with a question the worker can ask at the centre.

Everything else in this handbook is downstream of that idea.

## Where things live

| | |
|---|---|
| Code | `github.com/avinashnegi1999/yojana-sathi` (public) |
| Local checkout | `/run/media/avinash/Data/project Scheme Sathi` |
| Live bot | `@YojanaSathiBot` on Telegram |
| Host | AWS EC2, ap-south-1, systemd unit `sathi`, `/opt/sathi` |
| This handbook | `/run/media/avinash/Data/github/scheme-sathi-handbook` |

## Reading order

1. [01-the-problem/](01-the-problem/) — the half that matters most
2. [02-how-it-works/](02-how-it-works/) — the technical picture
3. [03-quality/](03-quality/) — testing, bugs, safety
4. [04-operations/](04-operations/) — running it
5. [05-assessment/](05-assessment/) — the honest score
6. [06-interview/](06-interview/) — what to say
7. [07-appendix/](07-appendix/) — glossary and file map

Next: [01-the-problem/01-who-this-is-for.md](01-the-problem/01-who-this-is-for.md)
