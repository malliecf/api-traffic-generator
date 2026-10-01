# api-traffic-generator

External synthetic traffic generator for the **api123.matthieuallie.net** widgets API.

## Why external?

Requests made from inside Cloudflare Workers to a hostname on the same zone are
dispatched internally: they hit the backend but never appear in the zone's
request logs (Log Explorer) or per-request analytics views. This generator runs
on GitHub Actions runners (public IPs outside Cloudflare), so every request
enters the zone as genuine public traffic and shows up in zone dashboards and
the Log Explorer.

## What it does

- GitHub Actions cron: two staggered schedules, every 5 minutes (00/05/10... and 02/07/12...)
- Each run: one burst of 100–300 requests, randomized pacing (120–400 ms)
- Endpoint mix: `GET /api/widgets` (~40%), `POST /api/widgets/1|2|3` (~20% each) with random widget JSON payloads
- Every request is marked with `User-Agent: api-traffic-generator/1.0` and an `X-Synthetic-Traffic` header

## Tuning

- Burst size / pacing: `BURST_MIN`, `BURST_MAX`, `INTERVAL_MIN`, `INTERVAL_MAX`, `TARGET_BASE` env vars in the workflow
- Manual run: Actions → **traffic** → **Run workflow** (set burst sizes)
- Pause: disable the **traffic** workflow in the Actions tab

## Notes

- The repo is public so GitHub Actions minutes are free; the code contains no secrets.
- Filter the zone Log Explorer by `User-Agent contains api-traffic-generator` to see exactly the generated requests.
