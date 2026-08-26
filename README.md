# uptime

Scheduled availability pings for services that would otherwise idle out.

## Why

Render's free tier spins a web service down after 15 minutes without an HTTP
request or a WebSocket message. Waking one costs roughly 22 seconds, measured
rather than estimated. When the service is linked publicly, that delay lands on
whoever opens the link first, which is exactly the wrong person to charge for it.

`.github/workflows/keepalive.yml` sends a request every 10 minutes, leaving a
spare firing inside each 15 minute window.

## What it pings

| Service | URL |
|---|---|
| collab-editor demo | https://crdt-editor-ivor.onrender.com |

## Two things worth knowing before trusting it

**GitHub's scheduled events are best-effort.** They can be delayed under load,
sometimes well past the 15 minute window this is defending. This reduces the
odds of a cold start; it does not remove them. A purpose-built uptime monitor
keeps time more reliably if that turns out to matter.

**The ping target is not arbitrary.** It has to be a real route the application
serves:

- Not `/robots.txt`, which Render answers itself while a service is asleep, so
  the request never reaches the service and never wakes it.
- Not `/health`, which reports on a Postgres that deployment deliberately does
  not have and answers 503 by design, so a perfectly healthy service would read
  as a failing one.

## The heartbeat file

GitHub disables scheduled workflows in a repository that has seen no activity
for 60 days. A pinger commits nothing by itself, so it would quietly switch
itself off after two months. The workflow writes `.heartbeat` once a day to keep
the repository active. That is the only reason the file exists.
