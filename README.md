# Your first RecruitMCP call in 5 minutes

Find backend engineers in Ghent from inside Claude or Cursor, with one config file and one question.

**[Get your free API key](https://recruitmcp.com/login?utm_source=github&utm_medium=tutorial&utm_campaign=first-call)**: 1,250 free credits. No card needed.


## What you need

- A RecruitMCP API key (step 1)
- Claude Desktop, Claude Code or Cursor
- For Claude Desktop only: Node.js, so `npx` works

## 1. Get your key (1 minute)

1. Open [recruitmcp.com/login](https://recruitmcp.com/login?utm_source=github&utm_medium=tutorial&utm_campaign=first-call).
2. Continue with GitHub, Google or email (you get a sign-in link by email).
3. Your first sign-in creates your workspace with 1,250 free credits and opens a **Your API key** popup. Copy the key right away. It starts with `rmcp_live_` and is shown only once.

Keep the key out of your code and out of git. Put it in an environment variable:

```bash
export RECRUITMCP_KEY="paste-your-key-here"
```

## 2. Add the MCP server (2 minutes)

The server lives at `https://mcp.recruitmcp.com/mcp` and takes your key as a Bearer token. Pick your client.

### Claude Code

Create `.mcp.json` in your project root:

```json
{
  "mcpServers": {
    "recruitmcp": {
      "type": "http",
      "url": "https://mcp.recruitmcp.com/mcp",
      "headers": {
        "Authorization": "Bearer ${RECRUITMCP_KEY}"
      }
    }
  }
}
```

Claude Code reads `${RECRUITMCP_KEY}` from your environment, so the key never lands in the file. Start Claude Code from the shell where you exported the key, then run `/mcp` to check that `recruitmcp` is connected.

### Cursor

Create `.cursor/mcp.json` in your project (or `~/.cursor/mcp.json` for all projects):

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

Cursor needs the `${env:NAME}` form. A plain `$RECRUITMCP_KEY` is sent as literal text and will not authenticate. Open Cursor Settings, then MCP, and check that `recruitmcp` shows its tools.

### Claude Desktop

Claude Desktop connects to remote servers through `mcp-remote`. Add this to `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "recruitmcp": {
      "command": "npx",
      "args": [
        "-y",
        "mcp-remote@latest",
        "https://mcp.recruitmcp.com/mcp",
        "--header",
        "Authorization: Bearer ${RECRUITMCP_KEY}"
      ],
      "env": {
        "RECRUITMCP_KEY": "paste-your-key-here"
      }
    }
  }
}
```

Restart Claude Desktop after saving. This file holds your key, so keep it private.

## 3. Ask for a people search (1 minute)

Type this in your client:

> Find backend engineers in Ghent, Belgium. Do a dry run first and show me the estimate, then show me 5 results.

Behind the scenes your client makes three tool calls:

1. **`resolve_location`** turns "Ghent, Belgium" into an id. Location filters take ids, not free text. You get `90009596`, the Ghent Metropolitan Area. This call is free.
2. **`search_people` with `dry_run: true`** previews the search and charges nothing:

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

3. **`search_people`** runs the same call without `dry_run`.

## 4. See your results (1 minute)

You get a short profile per person: `full_name`, current position, `location`, `profile_id` and `profile_url`. The response also has `cost.credits`, so you always see what a call spent.

From here, ask for full profiles of the people you like (`enrich_profiles`), save them to a list (`save_to_list`), or narrow the search with more filters.

## Prefer curl?

Same flow over the REST API.

Resolve the place (free):

```bash
curl -sS 'https://api.recruitmcp.com/v1/lookups/locations?q=Ghent&prefer_area=true' \
  -H "Authorization: Bearer $RECRUITMCP_KEY"
```

Preview the search (free):

```bash
curl -sS -X POST 'https://api.recruitmcp.com/v1/people/search?dry_run=true' \
  -H "Authorization: Bearer $RECRUITMCP_KEY" \
  -H 'content-type: application/json' \
  -d '{"filters":{"current_job_titles":["Backend Engineer"],"geo_ids":["90009596"]},"limit":5}'
```

Run it. A paid REST call needs an `Idempotency-Key` header, so a retry never charges twice:

```bash
curl -sS -X POST 'https://api.recruitmcp.com/v1/people/search' \
  -H "Authorization: Bearer $RECRUITMCP_KEY" \
  -H 'content-type: application/json' \
  -H "Idempotency-Key: first-call-$(date +%s)" \
  -d '{"filters":{"current_job_titles":["Backend Engineer"],"geo_ids":["90009596"]},"limit":5}'
```

The `X-Credits-Charged` response header shows what the call spent.

## Troubleshooting

- **401 or no tools in Cursor:** check that you used `${env:RECRUITMCP_KEY}` and that Cursor was started with the variable set.
- **No tools in Claude Code:** start it from the shell where you ran `export RECRUITMCP_KEY=...`, then check `/mcp`.
- **Missing a tool:** `tools/list` only shows the tools your key's scopes allow.

Full reference: [recruitmcp.com/docs](https://recruitmcp.com/docs?utm_source=github&utm_medium=tutorial&utm_campaign=first-call) and [recruitmcp.com/docs/mcp](https://recruitmcp.com/docs/mcp?utm_source=github&utm_medium=tutorial&utm_campaign=first-call).

## Ready?

**[Get your free API key](https://recruitmcp.com/login?utm_source=github&utm_medium=tutorial&utm_campaign=first-call)** and run your first search in the next 5 minutes. 1,250 free credits.
