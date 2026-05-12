# Google Ads MCP Server Setup

The skill uses Google's official Google Ads MCP server (`ads_mcp`) via stdio transport. Two tools:

- `execute_gaql` — run any GAQL query. Params: `query`, `customer_id`, `login_customer_id`.
- `list_accessible_accounts` — list all customer IDs accessible by your credentials.

## Install

```bash
pipx install ads-mcp
```

This puts the server in your pipx cache. Find the path:

```bash
find ~/.local/pipx/.cache -name "ads_mcp" -type d
```

You'll get something like `~/.local/pipx/.cache/<hash>/bin/python`. You need that path for the MCP config below.

## Google Ads credentials

The server reads `~/.google-ads.yaml`. Build the file using Google's [official guide](https://developers.google.com/google-ads/api/docs/oauth/cloud-project) — you'll need:

- A Google Cloud project with the Google Ads API enabled
- An OAuth 2.0 client ID + client secret
- A refresh token (generate via `google-oauthlib-tool`)
- A developer token from your Google Ads account
- Your login customer ID (the MCC, if you operate through one)

The yaml looks like:

```yaml
developer_token: "your_developer_token"
client_id: "xxxxxxxxxx.apps.googleusercontent.com"
client_secret: "your_client_secret"
refresh_token: "your_refresh_token"
login_customer_id: "1234567890"  # MCC if applicable
use_proto_plus: true
```

## Claude Code MCP config

Add to your project's `.claude/settings.json` under `mcpServers`:

```json
{
  "mcpServers": {
    "google-ads": {
      "command": "/Users/YOU/.local/pipx/.cache/<hash>/bin/python",
      "args": ["-m", "ads_mcp.stdio"],
      "env": {
        "GOOGLE_ADS_CONFIGURATION_FILE_PATH": "/Users/YOU/.google-ads.yaml"
      }
    }
  }
}
```

Replace `/Users/YOU/.local/pipx/.cache/<hash>/` with your real path. Replace `/Users/YOU/.google-ads.yaml` with your real yaml path.

Restart Claude Code. The `execute_gaql` and `list_accessible_accounts` tools become available.

## Verify

In Claude Code, ask: "List my accessible Google Ads accounts." If you get a list, you're set.

## Troubleshooting

| Symptom | Fix |
|---|---|
| Tools not available in session | The MCP config needs to be in the active project's `.claude/settings.json`. Restart after adding. |
| `pipx cache path changed` after re-install | Run the `find` command above, update the `command` path in settings.json. |
| Auth errors / 401 from Google | Refresh token expired or revoked. Re-auth with `google-oauthlib-tool`. |
| `developer_token` invalid | The token is per-Google-Ads-account, not per-MCC. Use the one tied to the account you're querying. |
| Customer ID errors | Pass `customer_id` WITHOUT dashes. `123-456-7890` becomes `1234567890`. |
