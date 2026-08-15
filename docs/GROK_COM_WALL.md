# Grok.com wall

Grok.com chat history is **not** readable as a filesystem and there is **no scrape API** for the hub.

## What works

1. **Read:** connector tools return hub status, left-off crumbs, lane list.  
2. **Write:** `continuity_closeout` appends a dated block to the left-off file and can fan out with `connect_after`. It must not write the identity or health files.  
3. **Sync:** `continuity_sync` / `hub sync` / `hub pulse` push crumbs **from disk to lanes**.

## What does not work

- Pulling the last N Grok.com messages into markdown automatically  
- Treating the Grok.com thread as SoT  
- “Just sync the chat” without a tool call  

If a future official export or connector event exists, that would be a new inbound adapter. Until then, the write tool **is** the inbound path.

## Connector hygiene

- Advertise `mcp` and `mcp:write` so writes are not treated as a mystery elevation.  
- Restart the local MCP origin after schema changes.  
- Keep the tunnel pointed only at the MCP port.  
- Token stays out of git.
