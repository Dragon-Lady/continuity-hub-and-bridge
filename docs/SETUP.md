# Setup — continuity hub and bridge

Neutral recreation notes. Swap paths for your machine.

## 1. Source of truth

Keep a small allowlist, for example:

| Role | Example path |
|------|----------------|
| Left-off (wins conflicts) | `CONTINUE_HERE.md` |
| Durable identity | `memory/MEMORY.md` |
| Small lane delta | `memory/cli/lane_state.md` |
| Optional host brief | `HOST.md` |

Private journals: watch **meta only** (exists, hash, mtime) or do not watch them at all on a public demo.

## 2. Hub state

All runtime lives under a gitignored `.hub/`:

- `revs.jsonl` — revision log  
- `lane_crumbs/` — per-lane wake files + `LANES.md`  
- `facts.json` / `handoffs.json` — bridge store (not a second brain)  
- `lane_crumbs/<host>/stickies_snapshot.json` — inbound host-note snapshot  

Never commit `.hub/` or `.env` tokens.

## 3. MCP origin

Local HTTP MCP on loopback, Bearer token in a mode-600 env file. Named tunnel or Tailscale only to that port. Restart the origin after tool-schema changes so Grok.com sees new tools.

Example scopes: `mcp`, `mcp:write`.

## 4. Pulse timer

See [`HUB_PULSE.md`](HUB_PULSE.md) and [`systemd/`](systemd/).

## 5. What “bidirectional” means here

| Direction | Automatic? |
|-----------|------------|
| Disk SoT → lane crumbs | Yes — `hub pulse` / timer |
| Host stickies → hub snapshot | Yes — read-only snapshot, not identity file |
| Grok.com chat → disk | **Only if** the connector calls `continuity_closeout` |
| Grok.com history scrape | **No** — not available |
| Auto post to X | **No** |
