# nolliworks-graph

A Claude Code plugin for the Nolliworks practice graph:

- **MCP connection** (`.mcp.json`): the remote server the app serves at `/mcp`. You sign in through Nolliworks in your browser and approve Claude on `/oauth/consent`. After that, Claude acts as you: it sees what you can see, and what it writes is recorded as yours.
- **Skill** `practice-graph`: what can be done on the graph and how to go about it. It covers reading a source into the inbox rather than straight onto the record, closing facts instead of deleting them, and phrasing gaps as statements about the record. It includes an ontology primer, phrasing rules and recipes.
- **Commands**: `/nolliworks-graph:why`, `/nolliworks-graph:impact`, `/nolliworks-graph:log-source`, `/nolliworks-graph:gate`.

## Install

```bash
claude plugin marketplace add tomasmillar/nolliworks-plugin
```

```bash
claude plugin install nolliworks-graph@nolliworks
```

The server URL defaults to the local dev server (`http://localhost:5173/mcp`). For production, set it before starting Claude Code:

```bash
export NOLLIWORKS_MCP_URL=https://<your app>/mcp
```

Then run `/mcp` in Claude Code, choose `nolliworks-graph`, and authenticate. Your browser opens Nolliworks: sign in, then press **Allow**.

**If you use an authenticator app:** after connecting, open `/oauth/verify` in the same browser and press **Confirm** next to the app. The database answers an app's session only once you've confirmed it past your second factor. Supabase Auth ends an app's session whenever you next sign in with your code somewhere else. When that happens, reconnect from `/mcp` and confirm again. See `docs/graph/mcp.md`.

## Claude.ai and Claude Desktop

Add a custom connector with the URL `https://<your app>/mcp`. The skill can be added separately: zip `skills/practice-graph` and upload it under Settings → Capabilities → Skills.

## Tools

| Tool | Does |
|---|---|
| `whoami` | You, and your practices |
| `list_projects`, `search`, `list_nodes`, `list_edges`, `get_node` | Read the graph |
| `describe_ontology` | The types, props, vocabularies and edges the database enforces |
| `why`, `impact` | The reasons behind a node; what rests on it |
| `stage_status` | Live stage, gate checks and findings |
| `list_inbox`, `add_source`, `propose`, `confirm_proposal`, `reject_proposal` | The inbox: what a model reads waits there for a person |
| `create_node`, `update_node`, `create_edge`, `update_edge`, `close_edge`, `delete_node` | Write as you, when you've said so |
