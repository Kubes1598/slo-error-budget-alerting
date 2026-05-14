# slo-error-budget-alerting

Multi-window, multi-burn-rate Prometheus alerts. The idea (taken almost verbatim from chapter 5 of the Google SRE workbook) is that you page when the error budget is actually at risk, not every time a metric crosses a threshold for five minutes.

I've been paged at 3am for a 30-second blip. So have you. This is the alternative.

## The SLO

```
99.9% availability over a 30-day rolling window
= 43 minutes 12 seconds of acceptable downtime per month
```

If the service is breaking the SLO at exactly 1× the budget rate, it lasts the full 30 days. Faster than 1× and the budget runs out early. The whole alerting system here just measures "how fast are we burning the budget right now?"

## The burn-rate thresholds

| Burn rate | Time to consume budget | Action       | Why                                            |
| --------- | ---------------------- | ------------ | ---------------------------------------------- |
| 14.4×     | 2 hours                | Page         | Whole month's budget gone in 2 hours.          |
| 6×        | 5 hours                | Ticket       | Real burn but there's time to think.           |
| 3×        | 10 hours               | Daily review | Trending wrong. Worth investigating soon.      |
| 1×        | 30 days                | Baseline     | Expected.                                      |

## Multi-window alerts

The page-level alert requires breach on both the 1-hour AND 5-minute window. The ticket-level alert requires both the 6-hour AND 30-minute window. Requiring two windows is what eliminates the brief-blip false positives. A 30-second spike doesn't move the 1-hour window enough to alert.

## What's in this repo

| File                      | What it is                                                  |
| ------------------------- | ----------------------------------------------------------- |
| `recording-rules.yml`     | Pre-compute error rates over 5m, 30m, 1h, 6h windows.       |
| `alert-rules.yml`         | Burn-rate alerts (page + ticket) consuming the rates.       |
| `alertmanager-config.yml` | Severity routing: page → PagerDuty, warning → Slack.        |
| `runbooks/`               | One-page docs linked from every alert annotation.           |

## Wiring it in

```yaml
# prometheus.yml
rule_files:
  - recording-rules.yml
  - alert-rules.yml
```

Per consumer service, set the SLO target label:

```yaml
- job_name: api
  static_configs:
    - targets: ["api:4000"]
      labels:
        slo_target: "0.999"
```

## Why runbooks matter

Every alert has a `runbook` annotation pointing to a one-pager. The first thing the responder sees in PagerDuty isn't "5xx rate exceeded threshold" — it's a link to the runbook that says "here's what to check first, here's the rollback command, here's who to call."

This is the difference between a 20-minute incident and a 90-minute one.

## Companion

- The Prometheus + Alertmanager runtime: [prometheus-grafana-stack](https://github.com/Kubes1598/prometheus-grafana-stack).
- The Helm chart whose metrics feed these rules: [k8s-helm-observability](https://github.com/Kubes1598/k8s-helm-observability).

## References

- Google SRE Workbook, ch.5: [Alerting on SLOs](https://sre.google/workbook/alerting-on-slos/)
- The specific multi-window pattern: [Multiwindow, Multi-Burn-Rate Alerts](https://sre.google/workbook/alerting-on-slos/#6-multiwindow-multi-burn-rate-alerts)

## License

MIT. See [LICENSE](./LICENSE).
