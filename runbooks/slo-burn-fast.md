# Runbook: Fast Error-Budget Burn (Pageable)

**Triggered by**: `ErrorBudgetBurn_Page` — service is burning the error budget at >14.4× over both the 1-hour and 5-minute windows. At this rate, the 30-day budget (43 min 12 sec of downtime) will be exhausted in 2 hours.

## First 5 minutes

1. **Acknowledge the page** in PagerDuty.
2. **Check the Grafana SLO dashboard** for the affected service:
   - Error rate over 5m/30m/1h
   - p99 latency
   - Recent deploys (annotations on the chart)
3. **Recent deploy?** Roll back immediately:
   ```
   gh workflow run rollback.yml -f service=<service> -f sha=<previous-sha>
   ```
4. **No recent deploy?** Check dependencies:
   - Upstream service status (database, auth, third-party)
   - Network — recent kubectl events, node pressure
   - DNS — sample resolution from a debug pod

## Triage signals

| Signal                              | Likely cause                          | First action                                           |
| ----------------------------------- | ------------------------------------- | ------------------------------------------------------ |
| 5xx rate jumped exactly at deploy   | Bad deploy                            | Rollback.                                              |
| 5xx rate climbed gradually          | Memory/connection leak                | Restart pods, then investigate. Save heap dump if Java. |
| Errors only from one pod            | Hot pod / bad scheduling              | Cordon node, drain pod.                                |
| Errors correlated with traffic spike | Capacity / autoscale lag             | Manually scale up. Check HPA metric source.            |
| All upstream calls failing          | Dependency outage                     | Open incident in dependency's channel.                 |

## Communication

- **#incidents** Slack channel: declare a SEV-2 if rollback works in <15m, otherwise SEV-1.
- Status page: update with "investigating".
- Stakeholders: tag the on-call manager if no progress in 30m.

## Closing the loop

- Once burn rate drops below 1× for 30 minutes, the alert auto-resolves.
- File a postmortem ticket within 24 hours.
- Add a learning to this runbook if the triage signal was novel.
