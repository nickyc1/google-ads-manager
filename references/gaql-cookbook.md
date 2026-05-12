# GAQL Cookbook

Common queries the skill uses. Drop these into `execute_gaql` with your customer ID.

Replace `<START>` and `<END>` with ISO dates (e.g., `2026-03-01` and `2026-03-31`).

## 1. Campaign performance, last 30 days

```sql
SELECT
  campaign.id,
  campaign.name,
  campaign.status,
  campaign.advertising_channel_type,
  metrics.cost_micros,
  metrics.impressions,
  metrics.clicks,
  metrics.conversions,
  metrics.conversions_value
FROM campaign
WHERE segments.date BETWEEN '<START>' AND '<END>'
  AND metrics.cost_micros > 0
ORDER BY metrics.cost_micros DESC
```

Divide `cost_micros` by 1,000,000 for dollars.

## 2. Search terms with conversions, last 14 days

```sql
SELECT
  search_term_view.search_term,
  campaign.name,
  ad_group.name,
  metrics.cost_micros,
  metrics.clicks,
  metrics.conversions,
  metrics.conversions_value,
  metrics.impressions
FROM search_term_view
WHERE segments.date BETWEEN '<START>' AND '<END>'
  AND metrics.clicks > 0
ORDER BY metrics.cost_micros DESC
LIMIT 500
```

Aggregate the rows by `search_term` in post-processing (one row per term-per-ad-group).

## 3. Ad creative performance, last 30 days

```sql
SELECT
  ad_group_ad.ad.id,
  ad_group_ad.ad.name,
  campaign.name,
  ad_group.name,
  metrics.cost_micros,
  metrics.clicks,
  metrics.conversions,
  metrics.conversions_value,
  metrics.ctr
FROM ad_group_ad
WHERE segments.date BETWEEN '<START>' AND '<END>'
  AND ad_group_ad.status = 'ENABLED'
ORDER BY metrics.conversions DESC
```

## 4. Geographic breakdown

```sql
SELECT
  campaign.name,
  geographic_view.country_criterion_id,
  metrics.cost_micros,
  metrics.clicks,
  metrics.conversions,
  metrics.conversions_value
FROM geographic_view
WHERE segments.date BETWEEN '<START>' AND '<END>'
  AND metrics.clicks > 0
ORDER BY metrics.conversions DESC
LIMIT 100
```

Country criterion IDs are ISO numeric. Common: 2840 = US, 2124 = CA, 2484 = MX, 2276 = DE, 2826 = GB, 2036 = AU, 2356 = IN, 2608 = PH.

## 5. Device breakdown by campaign

```sql
SELECT
  campaign.name,
  segments.device,
  metrics.cost_micros,
  metrics.clicks,
  metrics.conversions
FROM campaign
WHERE segments.date BETWEEN '<START>' AND '<END>'
  AND metrics.cost_micros > 0
```

## 6. Auction insights (impression share data)

```sql
SELECT
  campaign.name,
  metrics.search_impression_share,
  metrics.search_budget_lost_impression_share,
  metrics.search_rank_lost_impression_share
FROM campaign
WHERE segments.date BETWEEN '<START>' AND '<END>'
  AND campaign.advertising_channel_type = 'SEARCH'
```

`search_budget_lost_impression_share` close to 0 with strong profit-to-spend = candidate for budget increase. Close to 0 with weak profit-to-spend = nothing to scale into.

## 7. Quality Score trend (run weekly, log to a sheet)

```sql
SELECT
  ad_group_criterion.keyword.text,
  campaign.name,
  ad_group.name,
  ad_group_criterion.quality_info.quality_score,
  ad_group_criterion.quality_info.creative_quality_score,
  ad_group_criterion.quality_info.post_click_quality_score,
  ad_group_criterion.quality_info.search_predicted_ctr
FROM ad_group_criterion
WHERE ad_group_criterion.status = 'ENABLED'
  AND ad_group_criterion.type = 'KEYWORD'
  AND ad_group_criterion.quality_info.quality_score IS NOT NULL
```

Declining Quality Score means rising CPCs. Track week-over-week.

## GAQL gotchas

- GAQL does not support `OR` with parentheses in WHERE. Split into two queries.
- `segments.geo_target_country` doesn't work on `campaign` — use `geographic_view`.
- When using `ad_group_ad`, `campaign.name` must be in SELECT if used in WHERE.
- `search_term_view` returns one row per term-per-ad-group — aggregate in post-processing.
- `cost_micros` is in micros (millionths of account currency). Divide by 1,000,000 for dollars.
