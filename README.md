# CiteAura MCP — GEO & AI visibility for coding agents

Remote [Model Context Protocol](https://modelcontextprotocol.io) server for Generative Engine Optimization (GEO): an agent can pull your AI visibility audit, read the defects behind missing citations, precheck fix artifacts locally, and verify the live deployment afterwards.

- **Endpoint:** `POST https://citeaura.com/api/v1/mcp` (Streamable HTTP)
- **Auth:** OAuth 2.1 + PKCE, discovered from `https://citeaura.com/.well-known/oauth-protected-resource`; scoped bearer tokens `read`, `tickets:write`, `verify:run`
- **Agent onboarding:** `POST https://citeaura.com/api/v1/public/agent-connect` (no prior configuration)
- **Docs:** https://citeaura.com/docs#mcp-agents
- **Official MCP Registry id:** `io.github.500wango/citeaura`
- **Machine-readable index:** https://citeaura.com/llms.txt

## What an agent can do here

| Area | Tools |
|---|---|
| Discovery & diagnosis | `list_projects`, `get_project_overview`, `get_site_audit`, `get_citation_readiness`, `get_visibility_report`, `get_prompt_research` |
| Grounded fixes | `list_tickets`, `get_ticket_detail`, `list_generated_assets`, `get_generated_asset`, `get_brand_facts`, `update_ticket` |
| Precheck & live verification | `precheck_artifact` (offline check of HTML, JSON-LD or `llms.txt`), `trigger_verify`, `get_job_status`, `get_verify_result` |

Prompts: `fix_geo_tickets`, `verify_deployed_fixes`.
Resource templates: `citeaura://projects/{project_id}/overview`, `.../facts`, `.../assets/{path}`.

Every metric returned by the read tools drills down to the sampled prompt, the model and version, the sampling mode (parametric memory vs live web retrieval) and the timestamp, so an agent can quote evidence instead of a score.

## Connect

### Claude Code

```bash
claude mcp add --transport http citeaura https://citeaura.com/api/v1/mcp
```

### Codex CLI (`~/.codex/config.toml`)

```toml
[mcp_servers.citeaura]
url = "https://citeaura.com/api/v1/mcp"
```

### Cursor (`~/.cursor/mcp.json`)

```json
{
  "mcpServers": {
    "citeaura": {
      "url": "https://citeaura.com/api/v1/mcp"
    }
  }
}
```

### Claude.ai / ChatGPT custom connectors

Add `https://citeaura.com/api/v1/mcp` as a remote MCP connector. The client discovers the authorization server from the protected-resource metadata and runs the OAuth 2.1 + PKCE flow; no API key has to be pasted anywhere.

### Agent without a human in the loop

```bash
curl -X POST https://citeaura.com/api/v1/public/agent-connect
```

## Typical loop

1. `get_project_overview` and `get_site_audit` — find out why an AI answer does not cite the brand (blocked crawler, JS-only hydration shell, conflicting public facts).
2. `list_tickets` / `get_ticket_detail` — pick the engineering ticket with its diff and acceptance criteria.
3. `precheck_artifact` — validate the generated `robots.txt` line, JSON-LD block or `llms.txt` locally, before anything is deployed.
4. `trigger_verify`, then `get_job_status` and `get_verify_result` — re-probe the live URL and confirm the citation or crawler state actually changed.

## Scope and limits

- The server reads the workspace the token was granted for; `update_ticket` is the only write tool and requires the `tickets:write` scope.
- `precheck_artifact` runs entirely offline on the supplied text — it does not fetch your site.
- Verification jobs run against live public URLs; results carry the sampling mode and timestamp so a run can be audited later.
- Commercial hosted service with a free trial; paid plans from $79/mo. No key is bundled with this repository.

## Repository contents

- `server.json` — the manifest published to `registry.modelcontextprotocol.io`.
- `LICENSE` — MIT for the documentation and manifest in this repository. The hosted service itself is proprietary.
