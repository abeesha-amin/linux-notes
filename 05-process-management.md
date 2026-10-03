# Process Management & Job Control

## Viewing processes

| Command | Purpose |
|---|---|
| `ps` | Processes tied to your current shell only |
| `ps aux` | ALL processes, all users, detailed — the one you'll use most |
| `ps -u username` | Processes owned by a specific user |
| `pgrep name` | Quickly find PID(s) matching a process name |
| `top` | Live, continuously updating process view |
| `htop` | Improved version of `top` — color, scrolling, mouse support, tree view (`sudo apt install htop` if missing) |

## Reading `ps aux` / `top` columns

| Column | Meaning |
|---|---|
| USER | Who owns the process |
| PID | Unique process ID |
| %CPU / %MEM | Current CPU / RAM usage |
| VSZ / VIRT | Virtual memory size (reserved, not necessarily all in active use) |
| RSS / RES | Resident memory — actual physical RAM in use right now (more meaningful than VSZ) |
| TTY | Terminal attached (`?` = none — typical of background daemons) |
| STAT / S | Process state: `R`=running, `S`=sleeping, `D`=uninterruptible sleep (disk I/O), `Z`=zombie, `T`=stopped |
| START | When it started |
| TIME | Total CPU time actually consumed (not wall-clock time running) |
| COMMAND | The program and its arguments |

**htop extras:** `F5` toggles tree view (shows parent→child process relationships — useful for tracing how something got launched), `F9` kills a selected process.

## Avoiding grep matching itself

```bash
ps aux | grep sublime        # might match grep's own command line too
ps aux | grep [s]ublime      # bracket trick avoids self-matching
```

## Job control

| Command | Purpose |
|---|---|
| `command &` | Start a command directly in the background |
| `Ctrl + Z` | Suspend (pause) the current foreground process |
| `Ctrl + C` | Interrupt/stop the current foreground process (sends SIGINT) |
| `jobs` | List background/suspended jobs in this shell session, each with a job number |
| `bg %1` | Resume a suspended job, but in the background |
| `fg %1` | Bring a background/suspended job back to the foreground |

Typical flow:
```bash
sleep 300        # start something long-running
# Ctrl+Z          → suspends it
jobs              # confirm it's listed as job [1]
bg %1             # resume it in the background, terminal freed up
fg %1             # bring it back to foreground later if needed
```

## Killing processes — signals

```bash
kill -l            # list all available signal names/numbers
kill PID            # default: sends SIGTERM (15) — polite request to shut down
kill -2 PID          # SIGINT — same as Ctrl+C
kill -9 PID          # SIGKILL — immediate, forceful, no cleanup possible
kill -19 PID         # SIGSTOP — pause/freeze without terminating
pkill -9 processname  # kill by name instead of PID
```

**Signal escalation order, gentlest to harshest:**
```
SIGSTOP (19)  → pause, resumable
SIGINT  (2)   → polite interrupt (Ctrl+C)
SIGTERM (15)  → polite shutdown request (default)
SIGKILL (9)   → forced, immediate, no cleanup — last resort only
```

**Why SIGKILL should be a last resort:** well-behaved processes get a chance to close files and clean up on SIGTERM. A forced SIGKILL can leave corrupted files or orphaned locks behind — only reach for `-9` when a process is genuinely unresponsive.

## Why this matters for security

- Unusually high, sustained %CPU from an unfamiliar process is a classic sign of something like cryptomining malware.
- Large numbers of zombie processes can indicate buggy software or a resource-exhaustion attack.
- `htop`'s tree view helps trace how a suspicious process was actually launched (e.g., spawned by a browser or email client rather than a normal system service) — useful during an investigation.
- Freezing a suspicious process with `SIGSTOP` (rather than killing it outright) can preserve its state for analysis, instead of losing that information on termination.
