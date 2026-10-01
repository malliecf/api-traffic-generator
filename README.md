# api-traffic-generator

External synthetic traffic generator for the **api123.matthieuallie.net** widgets API.

## Why external?

Requests made from inside Cloudflare Workers to a hostname on the same zone are
dispatched internally: they hit the backend but never appear in the zone's
request logs (Log Explorer) or per-request analytics views. This generator runs
on GitHub Actions runners (public IPs outside Cloudflare), so every request
enters the zone as genuine public traffic and shows up in zone dashboards and
the Log Explorer.

## How it runs (fully automatic)

- Cron heartbeats: every minute (`* * * * *`, backup line at 2-59/5)
- Each fire starts a job that bursts continuously for ~55 minutes, then the
  next fire replaces it (concurrency: cancel-in-progress) — so traffic keeps
  flowing even when GitHub's scheduler fires late
- Bursts: 500–1000 requests each, 5 requests in parallel per tick
  (a 1000-request burst completes in ~2 min instead of timing out)
- Every request is marked with `User-Agent: api-traffic-generator/1.0` and an
  `X-Synthetic-Traffic` header

Manual: Actions → **traffic** → **Run workflow** (restarts the engine immediately,
optionally with custom burst sizes).

## Tuning

- `BURST_MIN` / `BURST_MAX` per burst (workflow env), `BATCH_SIZE` parallelism
- Pause: disable the **traffic** workflow in the Actions tab

## Notes

- Public repo so Actions minutes are free; code contains no secrets.
- Sustained volume at defaults is roughly 5–10 req/s — lower `BURST_MAX` if the
  backend should see less.
- Filter the zone Log Explorer by `User-Agent contains api-traffic-generator`
  to see exactly the generated requests.
