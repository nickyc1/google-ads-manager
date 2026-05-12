---
name: google-ads-manager
description: Profit-first Google Ads management playbook. Use to run weekly reviews, mine search terms for negatives, propose budget reallocations, and check campaigns against circuit-breaker safety rails. Triggers on "google ads review", "search term mining", "budget reallocation", "ppc weekly", or any audit-and-optimize task against a Google Ads account via the GAQL MCP.
---

# Google Ads Manager

A profit-first operating playbook for Google Ads. The agent reads your account via the Google Ads MCP, evaluates campaigns against the rules below, and proposes changes you approve before applying.

## North star

**Profit-to-spend ratio is the north star. Not ROAS. Not conversions. Not clicks.**

Google's reported ROAS is correlation, not causation. It claims credit for conversions that happen anyway through email, organic, and direct traffic. Your profit-to-spend ratio comes from your own first-party data joined to campaign UTMs — that's the source of truth.

The skill operates against a target band of **1.5x – 2.0x profit-to-spend** by default. Set your own target in `references/account-config.md` when you install.

## What the skill does

Three sub-workflows. The agent invokes the right one based on your prompt:

### 1. Weekly review
- Pull last 7 days of performance via GAQL
- Compute profit-to-spend by campaign (from your Looker/BigQuery export)
- Check every campaign against circuit breakers
- Identify search-term and negative-keyword opportunities
- Generate a weekly report from `templates/weekly-report.md`

### 2. Search term mining
- Pull the last 14 days of search terms via GAQL
- Cluster by n-gram
- Surface high-spend / low-conversion terms as negative candidates
- Surface high-CTR / low-impression terms as new exact-match candidates
- Output a change proposal you approve before applying

### 3. Budget optimization
- Pull profit-to-spend by campaign
- Rank campaigns into PUSH / SCALE / TEST / PAUSE tiers
- Propose budget shifts within the existing account total (no spend increase without explicit approval)
- Flag every recommendation with an incrementality caveat on repeat-heavy campaigns

## Required setup

- **Google Ads MCP** configured with your customer ID and login customer ID. See `references/mcp-setup.md`.
- **Profit data source** — a CSV export, BigQuery view, or Looker export that joins campaign UTM to your real profit (not Google-reported revenue). Without this, the skill falls back to Google-reported ROAS with a warning.
- **A populated `account-config.md`** — your target profit-to-spend ratio, daily budget, campaign-type splits, and any account-specific circuit breaker overrides.

## Core principles

### Layered campaign architecture

Modern ecommerce Google Ads accounts run four layers. Don't rely on one campaign type to do everything.

1. **Search (foundation)** — most control, most transparent data. Segment Brand / Non-Brand / Competitor.
2. **Shopping (Standard preferred)** — visibility into product-level and query-level performance. Use custom labels in Merchant Center to segment by performance tier.
3. **Performance Max (supplemental)** — useful for reaching YouTube, Display, Discover, Gmail. Always exclude brand terms. Watch for overlap with Search.
4. **Demand Gen / YouTube (strategic)** — for tentpole launches, not daily optimization.

### Circuit breakers (non-negotiable)

The agent enforces these automatically. Edit thresholds in `references/account-config.md`.

- Total account daily spend exceeds 130% of target → immediate Slack alert, no auto-changes
- Single campaign spends $500+ in a day with zero conversions → propose pause, alert human
- Conversions drop to zero across all campaigns for 6+ hours during business hours → conversion tracking is likely broken, halt all bid/budget changes
- Any landing page returns 404 → pause all campaigns sending traffic there
- Account profit-to-spend below 1.0x for 3 consecutive days → propose 25% spend reduction, escalate

### Bidding strategy default

Target ROAS (tROAS) when a campaign has 30+ conversions in the last 30 days. Maximize Conversions for new campaigns gathering data. Switch to tROAS after week 4.

Never adjust bid targets by more than 15-20% at a time on established campaigns. Wait 7 days between adjustments to let the algorithm stabilize.

### Match type philosophy

- **Broad match** — default for non-brand campaigns paired with smart bidding and strong conversion data
- **Phrase match** — when you need control over query structure
- **Exact match** — reserved for proven high-value keywords where you want isolated performance data

### Negatives, always

A robust negative keyword list is what separates a profitable non-brand campaign from a money pit. Review search terms weekly. Add new negatives ruthlessly. The skill's search-term mining workflow handles this.

## Incrementality is your blind spot

Google takes credit for conversions that would have happened anyway. You can't eliminate this — but you can size it and price it in.

- **Proxy metric:** report new vs. returning buyer ratio weekly. If a campaign is 80%+ returning, it's likely cannibalizing email/organic.
- **Geo-holdout tests:** pause campaigns in 2-3 comparable regions for 3-4 weeks. Compare total revenue (not Google-attributed) in test vs. control.
- **Google's Conversion Lift tool:** $5K min budget per experiment (used to be $100K+). Worth running on your highest-spend campaigns annually.
- **Default behavior of the skill:** every scaling recommendation on a repeat-heavy campaign gets an incrementality caveat in the proposal output.

## Google rep recommendations: handle with care

Google reps optimize for Google's revenue. You optimize for your profit. These often conflict.

- **Accept:** policy fix-ups, conversion tracking improvements, irrelevant-negative suggestions
- **Evaluate:** broad match conversion, budget increase suggestions, keyword additions
- **Reject:** auto-applied recommendations (always off), Display expansion on Search, "redundant" keyword removal, unsolicited PMax campaign creation

## Approval workflow

The skill operates in propose-then-approve mode by default:

1. Agent runs the analysis
2. Agent writes a change proposal using `templates/change-proposal.md`
3. Human reviews and approves (typically via Slack or chat)
4. Only then does the agent apply the change
5. After applying, agent confirms and notes the change for the next weekly review

The only exception is circuit-breaker triggers, where the agent acts first and notifies after.

## References

- `references/mcp-setup.md` — installing and configuring the Google Ads MCP
- `references/account-config.md` — your account-specific targets and overrides (you edit this)
- `references/data-pipeline.md` — how to wire up your profit-data source
- `references/gaql-cookbook.md` — common GAQL queries the skill uses
- `templates/weekly-report.md` — output format for weekly reviews
- `templates/change-proposal.md` — output format for proposed changes
