---
name: agent-bridge
description: "Build MCP bridge servers for cross-agent communication — Hermes ↔ Reasonix, multi-agent shared message queues and task boards via FastMCP SSE transport."
version: 1.0.0
metadata:
  hermes:
    tags: [mcp, multi-agent, bridge, reasonix, communication]
    related_skills: [native-mcp]
---

# Agent Bridge — Cross-Agent MCP Communication

Build a shared MCP server that lets multiple AI agents (Hermes, Reasonix, Claude Code, etc.) communicate through message queues and task boards.

## When to Use

- You need Hermes and Reasonix to collaborate on tasks
- You want a persistent shared workspace between agent sessions
- You need one agent to delegate work to another

## Architecture

```
Hermes ──→ [Bridge MCP Server :9876] ←── Reasonix
              ├─ send_message / read_messages
              └─ create_task / list_tasks / complete_task
```

Each agent connects via HTTP SSE transport. State is persisted to a JSON file.

## Quick Start

### 1. Start the bridge server

```bash
python mcp_bridge.py
```

The server is available at `http://localhost:9876/sse`. State is stored at `~/.hermes/shared/bridge_state.json`.

### 2. Configure Hermes

Add to `~/.hermes/config.yaml`:

```yaml
mcp_servers:
  agent-bridge:
    url: "http://localhost:9876/sse"
    timeout: 60
```

Or via CLI: `hermes mcp add agent-bridge --url http://localhost:9876/sse`

Restart Hermes or `/reload-mcp`.

### 3. Configure Reasonix

Add to `~/.reasonix/settings.json`:

```json
{
  "mcpServers": {
    "agent-bridge": {
      "type": "sse",
      "url": "http://localhost:9876/sse"
    }
  }
}
```

## Available Tools

| Tool | Purpose |
|------|---------|
| `send_message(content, to_agent, from_agent)` | Send instant message to another agent |
| `read_messages(agent_name, mark_read)` | Read unread messages for an agent |
| `create_task(title, description, assignee, from_agent)` | Create a task for another agent |
| `list_tasks(status, assignee)` | List tasks with optional filter |
| `get_task(task_id)` | Get full task details |
| `complete_task(task_id, result)` | Mark task done with result |
| `mark_in_progress(task_id)` | Mark task as in-progress |

## Common Patterns

### Hermes delegates to Reasonix

```
Hermes: create_task(title="Fix CakeShop i18n", assignee="reasonix")
→ Reasonix: list_tasks(assignee="reasonix", status="pending")
→ Reasonix: mark_in_progress(task_id)
→ Reasonix: ... does the work ...
→ Reasonix: complete_task(task_id, result="Fixed 12 controllers")
→ Hermes: list_tasks(status="done")  // check results
```

### Quick back-and-forth

```
Hermes: send_message(content="Check the new PR #42", to_agent="reasonix")
→ Reasonix: read_messages(agent_name="reasonix")
→ Reasonix: send_message(content="LGTM, merge it", to_agent="hermes")
→ Hermes: read_messages(agent_name="hermes")
```

## Server Script

See `references/mcp_bridge.py` for the full server implementation.

## Pitfalls

- **Hermes MCP tools may not appear mid-session**: The `mcp_agent_bridge_*` tools are discovered at session start. If they don't appear after config change, reconnect. In the meantime, use `execute_code` with `mcp.client.sse` to call tools directly.
- **Reasonix is TUI-only**: `npx reasonix code` requires a real terminal. It won't accept piped input. Start it via `Start-Process` on Windows.
- **Both agents must actively poll**: Messages and tasks are NOT pushed. Each agent must call `read_messages` / `list_tasks` to see new items.
- **Server must be running**: If both agents restart, the bridge may need restarting too. On Windows, start it as a detached background process: `subprocess.Popen(..., creationflags=DETACHED_PROCESS)`.
- **State file location**: Defaults to `~/.hermes/shared/bridge_state.json`. Both agents need read access to this directory.
- **Reasonix config format**: Uses `settings.json` with `mcpServers` key, NOT `config.yaml` like Hermes. Reasonix `mcp install` only works with registry servers, not custom URLs — must edit manually.
