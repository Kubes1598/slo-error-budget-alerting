# Runbook: Slow Error-Budget Burn (Ticket)

**Triggered by**: `ErrorBudgetBurn_Ticket` — service is burning the error budget at >6× over both the 6-hour and 30-minute windows. Real, but not pageable. You have working hours to fix it.

## Why it's a ticket and not a page

At 6× sustained burn, the 30-day budget is consumed in roughly 5 days. Plenty of time to debug carefully without waking anyone up.

## Within the working day

1. Open the SLO dashboard. Identify the 6h trend.
2. **Is the error rate increasing, flat, or decreasing?**
   - **Increasing** — escalate priority; this could become a fast burn.
   - **Flat at 6×** — a chronic issue. Likely a deploy you didn't notice degraded the service.
   - **Decreasing** — let it resolve. Use the time to file the postmortem.
3. Look at the diff vs the last week's baseline:
   - Which endpoint? Which status code?
   - Which client? (User-Agent or tenant label)
   - Any correlated metric — latency, queue depth, DB CPU?

## Common causes of slow burns

- A new release pushed an error path that wasn't covered by alerts (e.g., 4xx-but-actually-5xx).
- A client is retrying aggressively on a known bug.
- A dependency rotation that's silently failing every few requests.

## Don't roll back unless you have to

A slow burn is rarely a regression severe enough to warrant rollback. Find the specific contract or input that's broken and ship a targeted fix.

## Closing the loop

- File a ticket with the dashboard link, the diff, and the hypothesis.
- Assign to the team that owns the failing endpoint.
- Add an alert label for the specific failure mode if you find one.
