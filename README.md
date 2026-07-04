<div align="center">

# Airgraph Guide

**Persistent memory, shared knowledge graphs, and routed agent skills - over MCP.**

Give Claude Code, Codex, Hermes, Cursor, VS Code, or any MCP client a long-term
memory: a graph of nodes and edges your agents read and write across sessions,
with a live web canvas to watch it grow. Use the hosted MCP endpoint directly,
then optionally sync Airgraph-backed skills into each agent's local skill folder.

![MCP](https://img.shields.io/badge/MCP-Streamable_HTTP-5b50e0)
![Auth](https://img.shields.io/badge/Auth-OAuth_+_Bearer-2ea44f)
![Skills](https://img.shields.io/badge/Skills-Airgraph_routed-0f766e)

</div>

## Hosted Endpoint

Airgraph is a remote MCP server. For normal use, you do not run a local server or
copy a shared API key. Point your client at:

```text
https://whale-app-kfijw.ondigitalocean.app/mcp
```

## Choose Your Client

Airgraph uses the client you pick during setup. Do not guess from OAuth
`client_name`; write the right local config for the agent you are connecting.

| Client | Default auth | Local config |
| --- | --- | --- |
| Hermes | Bearer token | `$HERMES_HOME/config.yaml` |
| Claude Code | OAuth | `.mcp.json` or Claude's user MCP config |
| Codex | OAuth | `.codex/config.toml` or Codex's user MCP config |
| VS Code / Cursor / Claude Desktop | OAuth when supported | Their remote MCP connector UI |

For Hermes, do **not** add `auth: oauth` unless you specifically want Hermes'
loopback OAuth flow. Use the bearer-token config below.

## Quick Connect

### Claude Code

```bash
claude mcp add --transport http airgraph https://whale-app-kfijw.ondigitalocean.app/mcp
```

Claude Code will start the OAuth flow. Depending on the client version and
environment, Airgraph supports both current OAuth shapes:

- **Browser consent**: a browser opens, you log in to Airgraph, then click
  **Allow**.
- **Device code**: the client shows a short code and a verification URL; open
  that URL, enter the code, then click **허용**.

No token needs to be pasted for OAuth-capable clients.

### Hermes

Hermes currently works best with an Airgraph `agk_...` token. First create a
token in the Airgraph web app:

1. Open Airgraph.
2. Go to **Settings -> 에이전트 연결 토큰**.
3. Choose **Hermes** and create a token.
4. Copy the `agk_...` value once.

Then add Airgraph to your named Hermes profile:

```yaml
mcp_servers:
  airgraph:
    url: "https://whale-app-kfijw.ondigitalocean.app/mcp"
    enabled: true
    headers:
      Authorization: "Bearer ${AIRGRAPH_TOKEN}"

skills:
  external_dirs:
    - "airgraph-skills"
```

Start Hermes with the token in your environment:

```bash
export AIRGRAPH_TOKEN=agk_your_token
```

If you already see `auth: oauth` under `mcp_servers.airgraph`, remove it before
using the bearer config.

### Other Remote MCP Clients

Add a custom or remote MCP server with this URL:

```text
https://whale-app-kfijw.ondigitalocean.app/mcp
```

- **VS Code** - [Add to VS Code](https://insiders.vscode.dev/redirect/mcp/install?name=airgraph&config=%7B%22name%22%3A%20%22airgraph%22%2C%20%22type%22%3A%20%22http%22%2C%20%22url%22%3A%20%22https%3A//whale-app-kfijw.ondigitalocean.app/mcp%22%7D)
- **Cursor** - Settings -> MCP -> Add new MCP server -> paste the URL above.
- **Claude Desktop / Web** - Settings -> Connectors -> Add custom connector ->
  paste the URL, then log in when prompted.

## Login And Auth

Airgraph scopes every MCP request to the signed-in user. Your agent only sees
your Airgraph databases.

### OAuth

Airgraph exposes OAuth discovery metadata for clients such as Claude Code, Codex,
VS Code, Cursor, and Claude Desktop. It supports:

- **Authorization Code + PKCE** for clients that can open a browser and receive a
  callback.
- **Device Authorization Grant** for clients or environments that cannot receive
  a loopback callback.

Both flows mint user-scoped `ago_...` access tokens automatically. You usually do
not see or copy those tokens.

### Manual Token

Use this for Hermes, or for any client that does not support OAuth.

1. Open the Airgraph web app.
2. Go to **Settings -> 에이전트 연결 토큰**.
3. Create a new token and copy the `agk_...` value.
4. Configure your client with:

```http
Authorization: Bearer agk_your_token
```

Claude Code example:

```bash
claude mcp add --transport http airgraph https://whale-app-kfijw.ondigitalocean.app/mcp \
  --header "Authorization: Bearer agk_your_token"
```

## Project-Scoped Client Setup

Use these when the Airgraph MCP connection should live with one workspace rather
than your global user config.

### Claude Code

Claude Code project MCP config lives in `.mcp.json`. Synced project skills live
in `.claude/skills`.

```json
{
  "mcpServers": {
    "airgraph": {
      "type": "http",
      "url": "https://whale-app-kfijw.ondigitalocean.app/mcp"
    }
  }
}
```

Bearer fallback with an environment variable:

```json
{
  "mcpServers": {
    "airgraph": {
      "type": "http",
      "url": "https://whale-app-kfijw.ondigitalocean.app/mcp",
      "headers": {
        "Authorization": "Bearer ${AIRGRAPH_TOKEN}"
      }
    }
  }
}
```

Claude Code may ask you to approve project-scoped `.mcp.json` servers on first
use.

### Codex

Codex project MCP config lives in `.codex/config.toml` for trusted projects.
Synced project skills live in `.agents/skills`.

```toml
[mcp_servers.airgraph]
url = "https://whale-app-kfijw.ondigitalocean.app/mcp"
enabled = true
```

For OAuth-capable Airgraph servers, run a one-time login after adding the server:

```bash
codex mcp login airgraph
```

Bearer fallback with an environment variable:

```toml
[mcp_servers.airgraph]
url = "https://whale-app-kfijw.ondigitalocean.app/mcp"
enabled = true
bearer_token_env_var = "AIRGRAPH_TOKEN"
```

Codex only loads project `.codex/config.toml` after the project is trusted.

### Hermes

Hermes setup is profile-scoped. Airgraph intentionally does **not** sync skills
for the default Hermes profile; use a named profile such as `research`.

Profile config lives at:

```text
$HERMES_HOME/config.yaml
```

For the conventional root layout, that is usually:

```text
~/.hermes/profiles/research/config.yaml
```

Bearer token config:

```yaml
mcp_servers:
  airgraph:
    url: "https://whale-app-kfijw.ondigitalocean.app/mcp"
    enabled: true
    headers:
      Authorization: "Bearer ${AIRGRAPH_TOKEN}"

skills:
  external_dirs:
    - "airgraph-skills"
```

`skills.external_dirs` is required. If it is missing, Hermes will not sync or
load Airgraph skills for that profile.

If Hermes created a file like
`$HERMES_HOME/mcp-tokens/airgraph.client.json`, that is an OAuth client
registration from the older loopback flow. It is not needed for the bearer-token
setup.

## Airgraph Skill Sync

Airgraph can act as a skill store. A node with `타입=스킬` stores the canonical
skill body, and `그룹` decides which agent should receive a local stub.

Use these `그룹` labels:

| Agent target | Group label |
| --- | --- |
| Claude Code | `claude-code` |
| Codex | `codex` |
| Hermes named profile | `hermes:<profile>` |

Examples:

- A skill for Claude Code and Codex: `그룹 = ["claude-code", "codex"]`
- A skill for Hermes profile `research`: `그룹 = ["hermes:research"]`

Why sync at all? Agent clients build their always-on skill index from local disk
at session start. Airgraph sync materializes tiny `SKILL.md` stubs locally:

- The stub frontmatter (`name`, `description`) is what the agent indexes.
- The stub body points back to Airgraph and tells the agent to fetch the live
  skill body over MCP when the skill is used.

So the rule is:

```text
list = synced locally
detail = read live from Airgraph
```

Adding or renaming a skill needs a re-sync. Editing a skill body in Airgraph does
not require a re-sync.

Local skill directories:

| Agent target | Local skill directory |
| --- | --- |
| Claude Code | `.claude/skills` |
| Codex | `.agents/skills` |
| Hermes profile `research` | `~/.hermes/profiles/research/airgraph-skills` |

## Setup Helpers

If you are working from the Airgraph server repository, not this landing README
repository, the helper scripts write the client-specific config and create the
expected skill directories:

```bash
# Claude Code: .mcp.json + .claude/skills
node scripts/agent-setup.mjs --target claude-code \
  --url https://whale-app-kfijw.ondigitalocean.app/mcp

# Codex: .codex/config.toml + .agents/skills
node scripts/agent-setup.mjs --target codex \
  --url https://whale-app-kfijw.ondigitalocean.app/mcp

export AIRGRAPH_TOKEN=agk_...

# Hermes named profile: config.yaml + airgraph-skills.
# Hermes defaults to bearer token config.
node scripts/agent-setup.mjs --target hermes --profile research \
  --url https://whale-app-kfijw.ondigitalocean.app/mcp
```

Bearer fallback:

```bash
export AIRGRAPH_TOKEN=agk_...

node scripts/agent-setup.mjs --target claude-code --auth bearer \
  --url https://whale-app-kfijw.ondigitalocean.app/mcp

node scripts/agent-setup.mjs --target codex --auth bearer \
  --url https://whale-app-kfijw.ondigitalocean.app/mcp

node scripts/agent-setup.mjs --target hermes --profile research --auth bearer \
  --url https://whale-app-kfijw.ondigitalocean.app/mcp
```

Skill sync examples from the server repository:

```bash
AIRGRAPH_MCP_URL=https://whale-app-kfijw.ondigitalocean.app/mcp AIRGRAPH_TOKEN=agk_... \
  node scripts/agent-skill-sync.mjs --target claude-code

AIRGRAPH_MCP_URL=https://whale-app-kfijw.ondigitalocean.app/mcp AIRGRAPH_TOKEN=agk_... \
  node scripts/agent-skill-sync.mjs --target codex

AIRGRAPH_MCP_URL=https://whale-app-kfijw.ondigitalocean.app/mcp AIRGRAPH_TOKEN=agk_... \
  node scripts/agent-skill-sync.mjs --target hermes --profile research
```

Restart or open a new agent session after changing MCP config or syncing skills.

## For An AI Agent Setting This Up

1. Pick the target client: Hermes, Claude Code, Codex, or another remote MCP
   client.
2. For Hermes, ask the human to create an `agk_...` token in Airgraph settings,
   store it in `AIRGRAPH_TOKEN`, and write a bearer `Authorization` header. Do
   not write `auth: oauth` for Hermes by default.
3. For Claude Code, Codex, VS Code, Cursor, or Claude Desktop, add the hosted MCP
   URL with the client-specific config above and complete OAuth when prompted.
   If the client shows a device code, send the human to the verification URL
   shown by the client; if a browser opens, ask them to log in and click
   **Allow**.
4. If OAuth is unavailable for any other client, use an `agk_...` manual token
   and store it in `AIRGRAPH_TOKEN`, not directly in committed config.
5. For ambient skills, sync only the Airgraph skills routed to the target's
   `그룹` label.

## What Your Agent Gets

A typeless, property-based knowledge graph, every call scoped to the signed-in
user:

| Area | What |
| --- | --- |
| Databases & views | List and create databases; board, graph, calendar, and table views over the same nodes |
| Nodes | Upsert typeless nodes with free-form properties: text, number, date, select, status, URL, and more |
| Edges | Typed relations with provenance and confidence |
| Recall | Search nodes, get context around a node, read the emergent schema |
| Provenance & audit | Sources, review queue, and memory-event log |
| Your own databases | Connect your Supabase and query it next to the graph, with writes and DDL behind flags you control |

Tools appear as `mcp__airgraph__*` in compatible clients.

## How It Works

- **Remote MCP** over Streamable HTTP - one hosted endpoint, any compatible
  client.
- **OAuth 2.1 + PKCE** with Dynamic Client Registration and protected-resource
  discovery for OAuth-capable clients.
- **Device-code OAuth** for agents that cannot receive a callback.
- **Manual `agk_...` tokens** for Hermes and clients without OAuth support.
- **Per-user isolation** - tokens are hashed at rest and resolved to one
  Airgraph user before any tool call runs.
- **Skill routing** - Airgraph skill nodes are selected by `그룹` and mirrored as
  local stubs for each agent.

---

Built with the [Model Context Protocol](https://modelcontextprotocol.io).
