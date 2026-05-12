# Account configuration

Fill this in for your account. The skill reads these values when proposing changes.

## Identity

```
Account name:        [e.g. Acme Ecommerce]
Customer ID:         [10-digit, no dashes]
Login customer ID:   [your MCC, if applicable]
Currency:            USD
Timezone:            America/Chicago
```

## Targets

```
Profit-to-spend target band:   1.5x – 2.0x
Account daily budget:          $______
Repeat / new-buyer split:      ___% / ___%
```

## Circuit breakers (override defaults if needed)

```
Daily spend cap (% of target):                 130%
Single campaign zero-conversion spend cap:     $500
Conversions-zero-for-N-hours tracking break:   6
Account profit-to-spend floor (3-day):         1.0x
```

## Campaign type splits (target % of daily spend)

```
Non-Brand Search:         __%
Brand Search:             __%
Standard Shopping:        __%
Performance Max:          __%
Demand Gen / YouTube:     __%
```

## Reporting destinations

```
Weekly report goes to:    [Slack channel or email]
Change proposals go to:   [Slack channel or email]
Circuit-breaker alerts:   [Slack channel or DM, high priority]
```

## Profit data source

```
Type:           [BigQuery view / Looker export / CSV]
Location:       [URL or path]
Refresh:        [Daily at HH:MM, or live]
Join key:       [UTM_campaign / UTM_content / etc.]
```

If your profit data source is not yet wired up, the skill falls back to Google-reported revenue with a warning in every report.
