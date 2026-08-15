# Hub pulse

A pulse is one scheduled beat so seats do not wait for a human to remember `hub sync`.

## Command

```bash
hub pulse              # watch + inbound snapshot + sync (X skipped)
hub pulse --dry-run    # rev++ if dirty; do not run push adapters
hub pulse --no-stickies
```

Typical JSON fields:

- `watch.rev` / `watch.changed`  
- `inbound.stickies.count` (secret-scanned; bodies dropped on hit)  
- `sync.lanes` — primary lanes connected  
- `skipped` — includes `x` unless explicitly included  

## Timer

User units (no root):

```
~/.config/systemd/user/cottage-hub-pulse.service
~/.config/systemd/user/cottage-hub-pulse.timer
```

Calendars in the example units: **09:00, 15:00, 21:00** local, plus **5 minutes after boot**. `Persistent=true` so a missed beat runs when the machine is back.

```bash
systemctl --user enable --now cottage-hub-pulse.timer
systemctl --user list-timers cottage-hub-pulse.timer
journalctl --user -u cottage-hub-pulse.service -n 50
```

Example unit files: [`systemd/`](systemd/). Point `COTTAGE_ROOT` and `ExecStart` at your hub launcher.

## Non-goals

- Scraping Grok.com  
- Auto-merging stickies into the identity file  
- Auto X  
- Replacing a human closeout ritual (the pulse *amplifies* it)
