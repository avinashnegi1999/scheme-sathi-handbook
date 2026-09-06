# Running it

---

## Local

```bash
python3 check.py                    # everything, no install step
python3 -m sathi.main --telegram    # needs TELEGRAM_TOKEN in .env
python3 -m sathi.metrics.report     # impact dashboard → impact.html
```

Zero third-party dependencies. Python 3.11+.

---

## Live

| | |
|---|---|
| Host | AWS EC2, `ap-south-1` |
| Path | `/opt/sathi` |
| Service | systemd unit `sathi`, user `sathi` |
| Env | `/etc/sathi/sathi.env` |
| Cost | ~$10.50/month against $120 credit |
| Logs | `sudo journalctl -u sathi -f` |

> **`/opt/sathi` is a plain file copy, not a git checkout.** `git pull` there
> does nothing. Deployment is `rsync` then `systemctl restart sathi`.

---

## Deploying

```bash
rsync -az --exclude '.env' --exclude '*.db' --exclude '__pycache__' \
  sathi data tests check.py pyproject.toml \
  ubuntu@<host>:/tmp/sathi-new/

ssh <host> 'cd /tmp/sathi-new && python3 check.py'      # test ON the target first
ssh <host> 'sudo cp -a /opt/sathi /opt/sathi.bak-$(date +%F-%H%M)
            sudo rsync -a --delete /tmp/sathi-new/ /opt/sathi/
            sudo chown -R sathi:sathi /opt/sathi
            sudo systemctl restart sathi'
```

Run the suite **on the target** before swapping. The VM runs Python 3.12; the
laptop runs 3.11. That difference is exactly where a deploy-only failure hides.

---

## Verifying a deploy actually worked

`systemctl is-active` says the process exists, not that it is talking to
Telegram. The real check:

```bash
sudo ss -tnp | grep 149.154.16    # ESTAB to Telegram for the bot's PID
```

Plus fresh `session_start` rows in the event log.

> **Do not use `getUpdates` returning 200 as proof nothing else is polling.**
> Telegram preempts an existing long poll rather than always returning 409.
> That diagnostic has lied once already.

Never run two pollers on one bot token — they steal each other's updates. The
provisioning script refuses to build a second instance for this reason.

---

## Rolling back

```bash
sudo rsync -a --delete /opt/sathi.bak-<timestamp>/ /opt/sathi/
sudo systemctl restart sathi
```

Every deploy makes a timestamped backup first. Rollback is one command, which is
the only reason deploying at 11pm is acceptable.

---

## The deployment lesson

Deployment was parked for days behind an estimate that it needed a clear day.
It took **under an hour**, most of it waiting for an instance to boot.

Two vendors were tried first — Azure (stuck behind academic verification) and
Fly.io (stuck behind a card that declines international transactions). The wall
was never the vendor; it was the card. AWS worked not because the card problem
was solved but because an account with $120 of credit already existed. The
problem stopped mattering rather than being fixed.

> **Deploy before you polish.** The estimate that kept it parked was wrong by an
> order of magnitude — and that had already been learned once, on the previous
> project, and written down.

---

Next: [../05-assessment/11-scorecard.md](../05-assessment/11-scorecard.md)
