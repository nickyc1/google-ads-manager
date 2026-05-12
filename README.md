# google-ads-manager

A [Claude Code](https://claude.com/claude-code) skill that runs your Google Ads account on a profit-first playbook.

It pulls performance via the Google Ads MCP, evaluates campaigns against circuit-breaker safety rails, and proposes changes for you to approve before applying. Designed to operate on a weekly cadence with daily safety checks.

## Why this exists

Google Ads accounts get worse over time without active management. The default options either:

1. Burn money on PMax and call it "AI optimization"
2. Hire a $5K/mo agency that's running 30 accounts and can't possibly think about yours
3. Have a marketer spend 8+ hours a week pulling reports manually

This skill is option 4: an agent that does the weekly review, the search-term mining, and the budget reallocation analysis, and hands you a one-page proposal you approve in 5 minutes.

The opinion baked in: **profit-to-spend is the north star, not ROAS**. Google's reported ROAS claims credit for conversions that happen anyway. The skill expects your own first-party profit data joined to campaign UTMs.

## What it does

Three sub-workflows the agent invokes based on your prompt:

| Workflow | Trigger | Output |
|---|---|---|
| Weekly review | "weekly review", "ppc weekly" | Filled `weekly-report.md` with TL;DR, campaign tiers, circuit breakers, proposed changes |
| Search term mining | "mine search terms", "find negatives" | Negative keyword candidates + new-keyword candidates as a change proposal |
| Budget optimization | "reallocate budget", "where should I shift" | Budget shift proposal within current daily total, no spend increase without explicit approval |

## Requirements

- [Claude Code](https://claude.com/claude-code)
- **Google Ads MCP** configured — see [`references/mcp-setup.md`](references/mcp-setup.md)
- **Your own profit data source** — BigQuery, Looker, or CSV — see [`references/data-pipeline.md`](references/data-pipeline.md)
- A populated [`references/account-config.md`](references/account-config.md) with your target profit-to-spend, daily budget, and circuit-breaker thresholds

The skill falls back to Google-reported revenue if you don't have a profit source wired up, but every report will warn you that scaling decisions made on Google's numbers are unreliable.

## Install

```bash
git clone https://github.com/nickyc1/google-ads-manager.git ~/.claude/skills/google-ads-manager
```

Set up the MCP server using [`references/mcp-setup.md`](references/mcp-setup.md), then fill in your [`references/account-config.md`](references/account-config.md). Restart Claude Code.

## Usage

In Claude Code:

```
Run a weekly review on my Google Ads account.
Window: last 7 days.
```

```
Mine search terms for negatives on customer_id 1234567890.
Window: last 14 days.
```

```
Reallocate budget. P:S target is 1.8x.
```

## How the agent thinks

The full operating manual is in [`SKILL.md`](SKILL.md). Short version:

- **North star:** profit-to-spend ratio (1.5x – 2.0x target band by default)
- **Layered architecture:** Search foundation, Shopping for product transparency, PMax supplemental, Demand Gen strategic
- **Bidding default:** tROAS at 30+ conversions/30 days, Maximize Conversions for new campaigns
- **Match types:** broad with smart bidding for non-brand, exact for proven high-value
- **Negatives:** weekly review, ruthless additions
- **Bid adjustments:** max 15-20% at a time, 7-day cooldowns
- **Incrementality:** assumed blind spot, every scaling proposal on repeat-heavy gets a caveat

## Circuit breakers

The agent enforces these automatically (overrides in `account-config.md`):

- Total daily spend exceeds 130% of target → alert, no auto-changes
- Single campaign spends $500+ in a day with zero conversions → propose pause
- Conversions drop to zero across all campaigns for 6+ hours during business hours → halt all bid/budget changes
- Any landing page returns 404 → pause campaigns sending traffic there
- Account profit-to-spend below 1.0x for 3 consecutive days → propose 25% spend reduction

## Approval workflow

Default mode is propose-then-approve:

1. Skill runs analysis
2. Skill emits a change proposal
3. You review and reply `approve` / `approve [#]` / `reject [reason]`
4. Skill applies approved changes only
5. Skill confirms application and notes for next week

The only exception: circuit-breaker triggers, where the skill acts first and notifies after.

## Repo structure

```
google-ads-manager/
├── SKILL.md                              # the operating playbook
├── README.md
├── references/
│   ├── mcp-setup.md                      # Google Ads MCP install + config
│   ├── account-config.md                 # your targets and circuit breakers
│   ├── data-pipeline.md                  # how to wire your profit data source
│   └── gaql-cookbook.md                  # common GAQL queries the skill uses
└── templates/
    ├── weekly-report.md                  # weekly review output format
    └── change-proposal.md                # change proposal output format
```

## License

MIT — see [LICENSE](LICENSE).

Built by [Nick Christensen](https://github.com/nickyc1).
