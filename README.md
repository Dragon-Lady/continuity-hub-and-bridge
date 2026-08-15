# Continuity hub and bridge

Demo / setup notes for a **cottage-style continuity hub**: one markdown source of truth on disk, a small hub that revisions and secret-scans it, lane adapters that fan crumbs out, and one MCP surface so Grok.com can **read left-off** and **write a closeout** without dumping the whole vault.

This repository is documentation. It is **not** a live cottage and it does not contain personal files.

## What problem this solves

Agent seats forget. Chat history is not a brain. The hub keeps a short set of canonical markdown files as SoT and gives every seat the same left-off.

```
canonical markdown (SoT)
        │  watch / rev++ / secret scan
        ▼
      hub pulse   (a few times per day)
        │
        ├─► inbound crumb snapshot (stickies / lane notes — never the identity file)
        └─► outbound connect/sync  (wake crumbs per lane)
                │
                ▼
     Grok.com MCP  ←  read status / left-off
                   →  write closeout (allowlisted)
```

## Honest wall (Grok.com)

Grok.com **cannot scrape chats**. There is no API to pull the conversation into the hub.

Inbound from chat only happens when the connector **calls a write tool** (`continuity_closeout` or a small note tool). If you want that to be automatic, the model has to invoke the tool. The hub cannot go fetch the thread.

Outbound (cottage → crumbs / paste card / status) **can** run on a timer. That is the pulse below.

## Safety defaults

- Secret-scan before any envelope or snapshot body is stored.
- Never auto-publish private identity / health files.
- Never auto-post to X.
- Stickies / host notes are **crumbs**. Promoting them into the identity file is human-led.
- Journal binaries stay meta-only (hash/mtime), not body.

## Hub CLI (shape)

Local launcher example:

```bash
export COTTAGE_ROOT="$HOME/cottage"   # your SoT folder
export PYTHONPATH="$COTTAGE_ROOT"
hub status
hub pulse          # one bidirectional beat
hub sync           # rev++ if dirty + connect primary lanes
hub pull --lane cli
hub closeout --lane grok.com --left-off "…" --next "…"
```

`hub pulse` does:

1. `watch` — rev++ if canonical files changed  
2. inbound snapshot of optional host sticky notes (clipped, secret-scanned; does not write the host)  
3. `sync` — connect primary lanes and write wake crumbs  
4. skip X unless you pass an explicit include flag  

## Timer (a few times daily)

User systemd (no root). Units live in [`docs/systemd/`](docs/systemd/).

```bash
# 09:00 / 15:00 / 21:00 local + 5 minutes after login
install -m 644 docs/systemd/cottage-hub-pulse.service ~/.config/systemd/user/
install -m 644 docs/systemd/cottage-hub-pulse.timer   ~/.config/systemd/user/
systemctl --user daemon-reload
systemctl --user enable --now cottage-hub-pulse.timer
systemctl --user list-timers cottage-hub-pulse.timer
```

Manual beat anytime: `hub pulse`.

## Grok.com connector

One MCP URL (example: a named Cloudflare tunnel to `127.0.0.1:8792/mcp`).

| Tool | Direction | Notes |
|------|-----------|--------|
| `continuity_hub_status` | read | rev, dirty, primary left-off, commands — no file bodies |
| `continuity_lanes` | read / connect | list, pull, status, connect, sync |
| `continuity_closeout` | **write** | append dated closeout to the left-off file only — never identity/health |
| `continuity_sync` | house → lanes | same as `hub sync` |

Scopes to request on the connector: `mcp` and `mcp:write`.

Chat-side write is allowlisted. Context is not authorization. No shell, no repo writes, no TV.

## Recreate the layout

1. A folder of markdown (left-off file + identity file + small lane delta).  
2. Hub code that watches those paths, hashes them, secret-scans, and writes `.hub/` (rev log, envelopes, lane crumbs).  
3. One MCP server that exposes the table above.  
4. Optional user timer calling `hub pulse`.  
5. Grok.com custom connector pointed at the tunnel.

Do **not** commit live `.hub/` state, tokens, or personal journals into a public repo.

## Related

- Public synthetic vault (no personal files): [`continuity-showcase-vault`](https://github.com/Dragon-Lady/continuity-showcase-vault)

More detail: [`docs/SETUP.md`](docs/SETUP.md) · [`docs/HUB_PULSE.md`](docs/HUB_PULSE.md) · [`docs/GROK_COM_WALL.md`](docs/GROK_COM_WALL.md)
