# client-report-generator — setup guide

Generate a week-over-week performance report per client from one config
sheet — Meta Ads, Google Ads and GA4 in, an AI-written summary out through
Gmail and/or Slack.

## What you need

| Requirement | Where to get it |
|---|---|
| Gmail credential | n8n → Credentials → Gmail → Google OAuth |
| Google Sheets credential | n8n → Credentials → Google Sheets → Google OAuth |
| OpenAI credential | platform.openai.com → API keys |
| Meta access token | Meta for Developers → your app → Marketing API token |
| Google Ads tokens | Google Ads API → access token + developer token |
| GA4 access token | Google Analytics Data API → OAuth token |
| Slack bot token | api.slack.com/apps → Bot token (chat:write) |

The five provider tokens are set as **environment variables on your n8n
instance** (`META_ACCESS_TOKEN`, `GOOGLE_ADS_ACCESS_TOKEN`,
`GOOGLE_ADS_DEVELOPER_TOKEN`, `GA4_ACCESS_TOKEN`, `SLACK_BOT_TOKEN`) — the
workflow reads `$env.*`, so nothing secret lives in the template.

## Step 1 — Import

n8n → Workflows → Import from file → select `workflow.json`.

## Step 2 — Attach credentials

Attach Gmail, Google Sheets and OpenAI credentials to the nodes that ask
for them (credential picker on each node).

## Step 3 — Run the setup lane once

Press **play** on the orange **`SETUP — press play on this node once`**
trigger (bottom lane). It creates a Google Sheet named
**Weekly Client Reports** with two tabs and seed rows:

- `config` — one row per client: `client_id`, `client_name`, `enabled`,
  `src_meta_ads`, `src_google_ads`, `src_ga4`, `sink_gmail`, `sink_slack`,
  `recipients_email`, `slack_channel`, `meta_ad_account_id`,
  `google_ads_customer_id`, `ga4_property_id`.
- `reports` — the sent-report ledger: `dedupe_key`
  (`client_id__report_date`), `client_id`, `report_date`, `sinks_sent`,
  `sent_at`. It powers idempotent re-runs.

The last node prints `spreadsheet_id` — copy it.

## Step 4 — One node, two values

Open the **Workflow Config** node. Set `sheet_id` (from Step 3) and
`owner_email` (where the run digest and error alerts go). Every Sheets
node resolves its document from here — this is the only node you edit.

## Step 5 — Fill the config tab

One row per client. Set each `src_*`/`sink_*` cell to `yes` or `no` —
that cell alone decides whether a source is fetched or a sink is used for
that client. Fill the matching account-id columns for enabled sources and
the recipient columns for enabled sinks.

## Step 6 — Activate

Delete the bottom setup lane (optional), press **Test workflow** once to
watch the four-client demo run with zero credentials, then set the
workflow **Active**. It runs every Monday 09:00 (workflow timezone —
Settings → Timezone, shipped GMT).

Optional but recommended: Settings → **Error workflow** → pick this same
workflow — its `On workflow error` lane emails you if anything unhandled
ever fails.

## Weekly rhythm

- **Monday 09:00:** each enabled client's enabled sources are fetched,
  week-over-week deltas are computed (Code node — the AI writes prose
  around finished numbers only), the report is sent through the enabled
  sinks, and `reports` is upserted on `client_id + report_date` so a
  re-run never double-sends.
- **Digest:** every run ends with one digest — reports delivered, skipped,
  and alerts (a source outage, a missing recipient, a duplicate).

## Troubleshooting

| Symptom | Fix |
|---|---|
| Sheets nodes say "no document" | `sheet_id` not pasted into Workflow Config (Step 4) |
| A source never runs | Its `src_*` cell is not `yes` in the client's row |
| Digest shows a `failure` alert | The named source fetch failed — check its env token; other clients were unaffected |
| Digest shows `duplicate` | That report already exists in `reports` — delete the ledger row to allow a re-send |
| Run at a wrong hour | Workflow Settings → Timezone — set yours |
| AI node 401 | Check the OpenAI credential |
| Slack silent | `SLACK_BOT_TOKEN` unset on the instance, or `slack_channel` empty for the row |
