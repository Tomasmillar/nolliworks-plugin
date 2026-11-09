# Nolliworks plugins for Claude

Claude plugins for Nolliworks, practice management for architects.

| Plugin | What it does |
|---|---|
| [`nolliworks-graph`](plugins/nolliworks-graph) | Connects Claude to your practice graph through Nolliworks' MCP server: read jobs, trace why something is so and what rests on it, check stage gates, and propose additions to a job's inbox for a person to confirm. |

## Install (Claude Code)

```bash
claude plugin marketplace add tomasmillar/nolliworks-plugin
claude plugin install nolliworks-graph@nolliworks
```

Then set the server URL and sign in; see the [plugin's README](plugins/nolliworks-graph/README.md).

## Claude.ai and Claude Desktop

Add `https://<your app>/mcp` as a custom connector. The skill can be uploaded separately: zip `plugins/nolliworks-graph/skills/practice-graph` and add it under Settings → Capabilities → Skills.

---

This repository is published automatically from the Nolliworks source. Changes made here directly are overwritten.
