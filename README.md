# Your first RecruitMCP call in 5 minutes

RecruitMCP, the recruiting data MCP server, lets your AI client search people, read full profiles and find contact details. This repo is a copy-paste setup that takes you from signup to a first successful call: one free call, then one small real search. There is nothing to install or run: RecruitMCP is a hosted server at `https://mcp.recruitmcp.com/mcp`, and every config you need is below.

**[Get your free API key](https://recruitmcp.com/login?utm_source=github&utm_medium=tutorial&utm_campaign=first-call)**: free credits at signup, no card.

## What it costs

| Step | Tool | Cost |
|---|---|---|
| Check your account | `get_account_overview` | Free |
| Turn a place into an id | `resolve_location` | Free |
| Preview a search | `search_people` with `dry_run: true` | Free |
| Run a search with `limit: 5` | `search_people` | Paid, per person returned. The dry run shows the max cost first |

A person you already got in this workspace is not charged again. Full profiles and contact details are paid too, but you don't need either for your first call. Prices: [recruitmcp.com/pricing](https://recruitmcp.com/pricing?utm_source=github&utm_medium=tutorial&utm_campaign=first-call).

## 1. Sign up and copy your key (1 minute)

1. Open [recruitmcp.com/login](https://recruitmcp.com/login?utm_source=github&utm_medium=tutorial&utm_campaign=first-call) and continue with GitHub, Google or email.
2. Your first sign-in creates your workspace and shows **Your API key** once. Copy it now. It starts with `rmcp_live_`.
3. Lost it? Create a new key in the console under [API keys](https://recruitmcp.com/app/keys?utm_source=github&utm_medium=tutorial&utm_campaign=first-call), or use [recover](https://recruitmcp.com/recover?utm_source=github&utm_medium=tutorial&utm_campaign=first-call).

Keep the key out of your code and out of git. For the terminal clients below, put it in an environment variable:

```bash
export RECRUITMCP_KEY='rmcp_live_...'
```

**OAuth or API key?** Claude and ChatGPT connect with OAuth: the app opens a "Connect to RecruitMCP" page and you paste your key there. Cursor, Claude Code, Codex and curl send the key as a header: `Authorization: Bearer rmcp_live_...`. Either way you need the key from step 1.

## 2. Connect your client (2 minutes)

The server URL for every client is `https://mcp.recruitmcp.com/mcp`. Pick one. The steps below follow [MCP setup](https://recruitmcp.com/docs/mcp?utm_source=github&utm_medium=tutorial&utm_campaign=first-call), the source of truth for every client. If a step here differs from that page, follow the page.

### Claude (claude.ai and Claude Desktop)

1. In Claude, open **Customize > Connectors**, click **+ Add**, then **Add custom connector**.
2. Name: `RecruitMCP`. URL: `https://mcp.recruitmcp.com/mcp`. Click **Continue**.
3. Under Authentication choose **Sign in now**. If asked how Claude identifies itself, choose **Register automatically**. Leave request headers empty and click **Add**.
4. Click **Connect**, paste your API key on the Connect to RecruitMCP page, keep the permissions and click **Approve**. Claude gets a short-lived token, never your key.
5. In a chat, click **+**, open **Connectors** and switch RecruitMCP on.

A connector you add works on claude.ai and in Claude Desktop. On the Claude Free plan you can add one custom connector. On Team and Enterprise an Owner adds it once in **Organization settings > Connectors**, then each member clicks Connect.

### ChatGPT

Custom MCP servers in ChatGPT work on the web. ChatGPT's help pages list full support on Business, Enterprise and Edu. Pro connects read and fetch tools only. On Business, only admins and owners can add one.

1. In ChatGPT on the web, go to **Plugins**, select the plus button, then **Add custom MCP server**.
2. Name: `RecruitMCP`. Connection: `https://mcp.recruitmcp.com/mcp`. Authentication: **OAuth**.
3. Tick **I understand and want to continue**, then **Create as a plugin**.
4. Paste your API key on the Connect to RecruitMCP page, keep the permissions and click **Approve**.
5. Install the plugin, start a new chat, type **@**, pick RecruitMCP and ask your question.

### Cursor

Add this in Cursor MCP settings, or in `.cursor/mcp.json` in your project:

```json
{
  "mcpServers": {
    "recruitmcp": {
      "url": "https://mcp.recruitmcp.com/mcp",
      "headers": {
        "Authorization": "Bearer ${env:RECRUITMCP_KEY}"
      }
    }
  }
}
```

The entry is `url` plus `headers`; remote servers do not use `env` or `envFile`. Cursor reads `${env:RECRUITMCP_KEY}` from its own environment, so open Cursor from a terminal where `RECRUITMCP_KEY` is set, or paste the key into the header. A shell `$RECRUITMCP_KEY` in this file is left as text.

### Claude Code

In the terminal where `RECRUITMCP_KEY` is set:

```bash
claude mcp add --transport http recruitmcp https://mcp.recruitmcp.com/mcp --scope user --header 'Authorization: Bearer ${RECRUITMCP_KEY}'
```

The single quotes save a reference to the variable, so the key stays out of the Claude Code config.

### Codex

```bash
codex mcp add recruitmcp --url https://mcp.recruitmcp.com/mcp --bearer-token-env-var RECRUITMCP_KEY
```

Codex saves only the variable name in `~/.codex/config.toml`, never your key. Start `codex` in the same terminal and type `/mcp` to check that `recruitmcp` is listed.

### curl

No MCP client needed. Check your key with a free call:

```bash
curl -sS https://api.recruitmcp.com/v1/billing/balance \
  -H "Authorization: Bearer $RECRUITMCP_KEY"
```

The full curl flow is in step 3.

## 3. Your first two prompts (2 minutes)

### Prompt 1: free

> Use RecruitMCP to call get_account_overview. Tell me my credit balance and my key's scopes. Don't run a search.

This costs nothing. Optional extra: `get_guide` gives your client a longer introduction, also free.

### Prompt 2: a small paid search (limit 5)

> Use RecruitMCP to find 5 backend engineers in Ghent, Belgium. Resolve the location first, do a dry run and show me the estimate, then run the search with limit 5.

Your client makes three calls:

1. **`resolve_location`** (free) turns "Ghent, Belgium" into `90009596`, the Ghent Metropolitan Area. Location filters take ids, not free text.
2. **`search_people` with `dry_run: true`** (free) shows the estimate, including `credits_max`, the most the search can cost:

   ```json
   {
     "filters": {
       "current_job_titles": ["Backend Engineer"],
       "geo_ids": ["90009596"]
     },
     "limit": 5,
     "dry_run": true
   }
   ```

3. **`search_people`** without `dry_run` runs it. You pay only for people returned, never more than the dry run's `credits_max`.

### Same thing with curl

```bash
# 1. Resolve the place (free)
curl -sS 'https://api.recruitmcp.com/v1/lookups/locations?q=Ghent&prefer_area=true' \
  -H "Authorization: Bearer $RECRUITMCP_KEY"

# 2. Preview the search (free)
curl -sS -X POST 'https://api.recruitmcp.com/v1/people/search?dry_run=true' \
  -H "Authorization: Bearer $RECRUITMCP_KEY" \
  -H 'content-type: application/json' \
  -d '{"filters":{"current_job_titles":["Backend Engineer"],"geo_ids":["90009596"]},"limit":5}'

# 3. Run it (paid, limit 5; step 2 showed the max cost). A paid REST call needs an Idempotency-Key, so a retry never charges twice.
curl -sS -i -X POST 'https://api.recruitmcp.com/v1/people/search' \
  -H "Authorization: Bearer $RECRUITMCP_KEY" \
  -H 'content-type: application/json' \
  -H "Idempotency-Key: first-call-$(date +%s)" \
  -d '{"filters":{"current_job_titles":["Backend Engineer"],"geo_ids":["90009596"]},"limit":5}'
```

## It worked if you see...

- **Prompt 1:** your workspace name, a `balance_credits` showing your free credits, and scopes such as `sourcing:read` and `sourcing:spend`. `cost.credits` shows nothing was charged.
- **Dry run:** `"dry_run": true`, an `estimate` with `credits_max` and `"sufficient": true`, and `cost.credits` shows nothing was charged.
- **Search:** up to 5 people, each with `full_name`, `location`, `profile_id`, `profile_url` and `current_positions[]` (with `title` and `company_name`). `total_elements` shows how many people match in total. `cost.credits` is never more than the dry run's `credits_max`. With curl, the `X-Credits-Charged` header shows the same number.
- Run Prompt 1 again: your balance went down by exactly that number.

Next: ask for full profiles of the people you like (`enrich_profiles`, paid per profile found), save them with `save_to_list` (free), or narrow the search with more filters.

## Troubleshooting

| You see | What it means | Fix |
|---|---|---|
| `401 unauthorized`, "Missing bearer API key" | No key reached the server | Cursor: use `${env:RECRUITMCP_KEY}` and start Cursor from the terminal where it is set. Claude Code and Codex: start them from that terminal. |
| `401 unauthorized`, "Invalid or expired API key" | The key is mistyped, revoked, or still a placeholder | Paste a live key that starts with `rmcp_live_`, or create a new one under [API keys](https://recruitmcp.com/app/keys?utm_source=github&utm_medium=tutorial&utm_campaign=first-call). |
| `403` with `insufficient_scope` | The key or connection lacks the scope in `details.required_scope` | Create a key with that scope, or reconnect and keep the permissions on the Connect to RecruitMCP page. |
| `402 insufficient_credits` | The balance is below the estimate. Nothing ran | Run `get_account_overview`. Signing up in the console needs no extra step. Only if you created your workspace through the API (`POST /v1/workspaces`), confirm your email first: your free credits arrive after you open the link. |
| `approval_required` with a `confirm_token` (HTTP 409 on REST) | The estimate is above your key's approval threshold. Nothing was charged | Check the estimate. To go ahead, send the same call again with that `confirm_token`. |
| `400 idempotency_key_required` | A paid REST call without an `Idempotency-Key` | Add the header, or use `?dry_run=true` to preview. |
| `406 Not Acceptable` on a curl call to the MCP URL | The Accept header is missing | Send `Accept: application/json, text/event-stream`. |
| A tool is missing | The tool list only shows what your key's scopes allow | Check scopes with `get_account_overview`. Add `?toolsets=all` to the URL for the advanced tools. |
| `400 invalid_request`, "Location id ... is not numeric" | Location filters take ids, not place names | Call `resolve_location` first and pass the `geo_id` it returns. |

Full reference: [recruitmcp.com/docs](https://recruitmcp.com/docs?utm_source=github&utm_medium=tutorial&utm_campaign=first-call), [MCP setup](https://recruitmcp.com/docs/mcp?utm_source=github&utm_medium=tutorial&utm_campaign=first-call) and [error fixes](https://recruitmcp.com/docs/errors?utm_source=github&utm_medium=tutorial&utm_campaign=first-call).

## License

MIT. See [LICENSE](LICENSE).
