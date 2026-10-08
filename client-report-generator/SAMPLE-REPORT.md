# Sample report — client-report-generator

Real output of the template's zero-credential demo run
(`fixtures/config-v11.json` — anonymized demo clients only). Each client
gets this branded HTML report by Gmail (`delivery_mode: send` or `draft`)
and/or a markdown variant on Slack; the operator gets the run digest.

## Email 1 — Acme Retail — weekly report (2026-10-08)

- `client_id`: `acme` · `delivery_mode`: **draft** (filed to Gmail drafts
  for owner review) · `alert_threshold_pct`: 15 · brand: `#0b5fff` + logo

Markdown body sent with the branded HTML:

# Acme Retail — week ending 2026-10-08

| source | metric | this week | last week | WoW |
| --- | --- | --- | --- | --- |
| google_ads | spend | 840 | 700 | 20% FLAGGED |
| google_ads | conversions | 45 | 50 | -10% |
| ga4 | sessions | 1240 | 1100 | 12.7% |
| ga4 | conversions | 31 | 30 | 3.3% |

Flagged anomalies (|WoW| >= 15%):
- google_ads spend: 20% WoW

Acme Retail: this week spend up 20%; sessions up 12.7%; conversions up 3.3%; watch conversions down 10%. Flagged anomalies: google_ads/spend 20%. (demo prose — no AI call)

— weekly client report

## Email 2 — Boutique Hotels — weekly report (2026-10-08)

- `client_id`: `boutique` · `delivery_mode`: **send** ·
  `alert_threshold_pct`: 25 · brand: `#c2185b`, no logo

# Boutique Hotels — week ending 2026-10-08

| source | metric | this week | last week | WoW |
| --- | --- | --- | --- | --- |
| meta_ads | spend | 210 | 300 | -30% FLAGGED |
| meta_ads | clicks | 120 | 150 | -20% |
| google_ads | clicks | 320 | 280 | 14.3% |
| google_ads | conversions | 22 | 20 | 10% |
| ga4 | sessions | 640 | 640 | 0% |

Flagged anomalies (|WoW| >= 25%):
- meta_ads spend: -30% WoW

Boutique Hotels: this week clicks up 14.3%; conversions up 10%; watch spend down 30%; clicks down 20%. Flagged anomalies: meta_ads/spend -30%. (demo prose — no AI call)

— weekly client report

## Operator digest (one per run)

```
{
  "status": "digest",
  "summary": "Run complete: 2 report(s) delivered, 0 skipped, 0 alert(s), 2 anomaly flag(s)",
  "reports_delivered": 2,
  "skipped": 0,
  "anomaly_flags": 2,
  "flagged_metrics": [
    "acme: google_ads/spend 20%",
    "boutique: meta_ads/spend -30%"
  ],
  "needs_attention": []
}
```

The `reports` ledger row keyed `client_id + ISO week` means a re-run in
the same week is skipped as a duplicate — no double-sends.
