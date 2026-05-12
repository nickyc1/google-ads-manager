# Profit data pipeline

Google's reported ROAS and conversion value are correlation, not causation. To run this skill profit-first, you need your own data source that joins campaign spend (from Google Ads) to actual profit (from your real revenue and cost data).

This doc describes the contract. The implementation depends on your stack.

## The contract

The skill expects a table or view with at least these columns, refreshed daily:

| Column | Type | Example |
|---|---|---|
| `date` | date | `2026-04-15` |
| `campaign_name` | string | `Search - Non-Brand - AI Tools - tROAS` |
| `campaign_id` | string | `19283746521` (Google Ads campaign ID) |
| `utm_source` | string | `google` |
| `utm_medium` | string | `cpc` |
| `utm_campaign` | string | matches `campaign_name` or a derived label |
| `spend_usd` | numeric | spend in USD for the day |
| `revenue_usd` | numeric | true revenue captured from your own system |
| `profit_usd` | numeric | revenue minus COGS, refunds, fees, partner payouts, whatever you exclude |
| `orders` | integer | true order count from your own system |
| `new_buyer_orders` | integer | orders flagged as first-time buyers |

The skill reads from this table to compute profit-to-spend per campaign.

## Common implementations

### BigQuery view (recommended)

If your e-commerce data already lands in BigQuery (Shopify, Stripe, internal warehouse), build a view that joins:

- `orders` (from your platform)
- `ad_spend_daily` (from a Google Ads → BigQuery Data Transfer service, or a sync tool like Stitch/Fivetran)
- A UTM-to-campaign mapping (often a derived table)

Grant Claude Code read access via the BigQuery MCP or a service account.

### Looker export

If you already run Looker, build a Look with the columns above. Schedule a daily CSV export to Drive or S3. The skill reads the CSV.

### CSV from a manual pull

For small accounts, a daily CSV export from your e-commerce admin (Shopify, Stripe Sigma, etc.) plus a join against Google Ads UTMs is enough. Keep the file at a stable path the skill can read.

## UTM hygiene matters

The join is only as good as your UTMs. Some rules:

- Use auto-tagging in Google Ads (the `gclid` parameter). It's more reliable than manual UTMs for clickthroughs, but the manual UTMs are what your analytics joins on.
- Use a consistent `utm_campaign` value that matches your Google Ads campaign name OR a derived label you control on both sides.
- Avoid spaces and unusual characters in UTMs — they get URL-encoded inconsistently.
- Audit your UTM dictionary quarterly. Bad UTMs == bad profit data == bad decisions.

## What to do if you don't have this yet

The skill still works without a real profit source. It just falls back to Google-reported revenue and prepends every report with:

> ⚠️ Profit data source not configured. Numbers below are Google-reported, not real profit. Wire up a profit pipeline before scaling decisions.

You can run the skill for 2-4 weeks on Google-reported data to get a feel for it, but every scaling proposal needs a human-judgment caveat until real profit data is connected.

## How the skill uses this data

For each campaign:

1. Sum `spend_usd` and `profit_usd` over the analysis window
2. Compute `profit_to_spend = profit_usd / spend_usd`
3. Compare against the target band in `account-config.md`
4. Roll up to account level for the daily and weekly health checks
5. Compare new vs. returning buyer ratios for the incrementality proxy metric
