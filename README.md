<div align="center">

# Airgraph

**Persistent memory & a shared knowledge graph for your AI agents — over MCP.**

Give Claude, Cursor, or any MCP client a long-term memory: a graph of nodes and
edges your agents read and write across sessions, with a live web canvas to
watch it grow. Fully hosted — nothing to install, no API key to copy.

![MCP](https://img.shields.io/badge/MCP-Streamable_HTTP-5b50e0)
![Auth](https://img.shields.io/badge/Auth-OAuth_2.1_+_PKCE-2ea44f)

</div>

## Connect in 10 seconds

Airgraph is a **remote MCP server**. You don't download or run anything — point
your client at one URL and log in.

**Claude Code**

```bash
claude mcp add --transport http airgraph https://whale-app-kfijw.ondigitalocean.app/mcp
```

A browser opens → log in → click **Allow**. That's it. No token to paste.

**Any other client** (Cursor, VS Code, Claude Desktop/Web, …) — add a custom /
remote MCP server with this URL:

```
https://whale-app-kfijw.ondigitalocean.app/mcp
```

- **VS Code** — [**Add to VS Code →**](https://insiders.vscode.dev/redirect/mcp/install?name=airgraph&config=%7B%22name%22%3A%20%22airgraph%22%2C%20%22type%22%3A%20%22http%22%2C%20%22url%22%3A%20%22https%3A//whale-app-kfijw.ondigitalocean.app/mcp%22%7D)
- **Cursor** — Settings → MCP → *Add new MCP server* → paste the URL above (or open this deep link: `cursor://anysphere.cursor-deeplink/mcp/install?name=airgraph&config=eyJ1cmwiOiAiaHR0cHM6Ly93aGFsZS1hcHAta2Zpancub25kaWdpdGFsb2NlYW4uYXBwL21jcCJ9`)
- **Claude Desktop / Web** — Settings → Connectors → *Add custom connector* → paste the URL → log in when prompted.

> 🔐 **Zero-copy auth.** Connecting runs OAuth 2.1 (PKCE): a browser opens, you
> log into Airgraph and approve the agent. After that the agent only ever sees
> **your** databases. No shared secrets, nothing to copy or rotate.

## For an AI agent setting this up

If you're an agent reading this to connect yourself: run the Claude Code command
above (or your client's MCP-add equivalent) with transport `http` and URL
`https://whale-app-kfijw.ondigitalocean.app/mcp`, then ask the user to finish the
one-time browser login and approve access.

## What your agent gets

A typeless, property-based knowledge graph, every call scoped to the signed-in user:

| Area | What |
|------|------|
| Databases & views | list / create databases, board·calendar·table views |
| Nodes | upsert typeless nodes (free-form properties: text/number/date/select/status/url), move/place |
| Edges | typed relations with provenance & confidence |
| Recall | search nodes, get context around a node, read the emergent schema |
| Provenance & audit | sources, review queue, memory-event log |

Tools appear as `mcp__airgraph__*` in your client.

## See it

Your agent writes to the graph; you watch it draw itself on the web canvas in
real time.

<!-- ![demo](docs/demo.gif) -->
_(demo coming soon)_

## How it works

- **Remote MCP** over Streamable HTTP — one hosted endpoint, any client.
- **OAuth 2.1 + PKCE** with Dynamic Client Registration (RFC 7591) and
  protected-resource discovery (RFC 9728) — the agent does the whole handshake;
  you just log in once.
- **Per-user isolation** — access tokens are short-lived, hashed at rest, and
  bound to this server (audience-checked).

<details>
<summary><b>Manual token</b> (for clients without OAuth)</summary>

1. Open the Airgraph web app → **Settings → 연결 토큰 → 새 토큰 발급** → copy the `agk_…` token.
2. Add the server with an `Authorization` header:

   ```bash
   claude mcp add --transport http airgraph https://whale-app-kfijw.ondigitalocean.app/mcp \
     --header "Authorization: Bearer agk_your_token"
   ```

</details>

---

Built with the [Model Context Protocol](https://modelcontextprotocol.io).
