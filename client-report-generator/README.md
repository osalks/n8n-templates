# client-report-generator

Weekly client reports driven by a Google Sheet config — Meta Ads, Google
Ads and GA4 metrics, deterministic week-over-week deltas, AI narrative,
branded HTML delivery by Gmail (send or draft-for-review) and/or Slack,
an idempotent reports ledger, and one operator digest per run.

![Workflow canvas](screenshots/canvas.png)

## Overview

- 87-node n8n workflow, importable `workflow.json` — the demo lane runs
  four clients end-to-end with **zero credentials** (`Test workflow`).
- One row per client in a `config` sheet tab toggles sources, sinks,
  delivery mode (`send`/`draft`), brand header and the anomaly threshold —
  you edit cells, never the workflow.
- The AI only narrates finished numbers: WoW deltas and anomaly flags are
  computed in a Code node, so the prose can never invent math.

## Architecture

```
┌────────┐──▶┌────────┐──▶┌───────┐──▶┌───────────┐──▶┌────────┐──▶┌────────┐
│ config │──▶│ expand │──▶│ fetch │──▶│ normalize │──▶│ deltas │──▶│ dedupe │
└────────┘──▶└────────┘──▶└───────┘──▶└───────────┘──▶└────────┘──▶└───┬────┘
                                                                       │
             ┌────────┐◀──┌────────┐◀──┌───────┐◀──┌────────┐◀──┌──────▼────┐
             │ digest │◀──│ ledger │◀──│ sinks │◀──│ render │◀──│ narrative │
             └────────┘◀──└────────┘◀──└───────┘◀──└────────┘◀──└───────────┘
```

Three layers: the `config` sheet is the control plane; per-source
fetch+normalize adapter pairs sit behind `live?` gates (a named extension
dock attaches new sources/sinks without engine changes); the engine is
config-agnostic — dedupe on `client_id + ISO week`, render, fan out to
sinks, upsert the `reports` ledger, close with an operator digest.

## What it does

Every Monday 09:00 it reads the `config` tab, validates each row, expands
one fetch per enabled source, normalizes to one metric contract, computes
`delta_pct` deterministically and flags metrics past the client's
`alert_threshold_pct`, skips clients already reported this ISO week,
writes the narrative, renders a per-client branded HTML report
(`brand_name`/`brand_color`/`brand_logo_url` cells), delivers through the
enabled sinks (Gmail send **or** draft-for-review via `delivery_mode`,
Slack with `ok`-flag verification), records the send in the `reports`
ledger, and emails the operator a run digest. A source outage alerts
instead of blocking other clients; an unhandled failure lands in the
error lane and is emailed to the operator.

See a real rendered report: [SAMPLE-REPORT.md](SAMPLE-REPORT.md)
(produced by the committed fixture run — anonymized demo clients only).

## Setup

Step-by-step: **[SETUP.md](SETUP.md)** — import `workflow.json`, press
play on the SETUP trigger to create the workbook, paste the printed
`spreadsheet_id` into **Workflow Config**, attach credentials (Gmail +
Sheets + OpenAI, plus Header Auth / Custom Auth on each HTTP node), set
the operator address, fill the `config` tab, activate.

## Use cases

- Agencies sending the same weekly performance report to every client.
- Freelancers who want draft-for-review before anything goes to a client.
- Marketing ops that want anomaly flags and an audit ledger, not just a
  pretty email.

## Custom builds

This template is a starting point — extra sources, extra sinks, different
cadence, approval steps, CRM or warehouse writes. If you want it adapted
or a bespoke workflow built for your stack, email **contact@osalk.com**
or visit [osalk.com](https://osalk.com).
