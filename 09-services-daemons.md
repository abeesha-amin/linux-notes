# Processes, Daemons, Services & systemd

## Clarifying the terminology

These three terms describe the same underlying thing at different levels of abstraction:

```
systemd (the manager)
    │
    └── manages → SERVICES (defined, controllable units)
                       │
                       └── implemented as → DAEMONS (background processes)
                                                 │
                                                 └── technically → PROCESSES (any running program)
```

- **Process** — any running program, kernel-level concept. Visible via `ps aux`.
- **Daemon** — a background process with no attached terminal, runs continuously, usually starts at boot. Conventionally named ending in "d" (`sshd`, `snapd`, `systemd`).
- **Service** — the administrative concept: a daemon that's been formally registered with `systemd`, so it can be consistently started/stopped/monitored/configured.

## systemctl — managing services

```bash
systemctl status servicename      # is it running? enabled? what state?
sudo systemctl start servicename   # start now
sudo systemctl stop servicename    # stop now
sudo systemctl restart servicename # stop then start — needed after config changes
sudo systemctl enable servicename  # auto-start on every future boot
sudo systemctl disable servicename # stop auto-starting on boot
systemctl list-units --type=service               # everything systemd knows about
systemctl list-units --type=service --state=running  # only what's currently active
```

### Reading `systemctl status` output
```
Loaded: loaded (/usr/lib/systemd/system/ssh.service; disabled; ...)
Active: inactive (dead)
```
- `Loaded: loaded` — the service **is installed**, systemd knows about it
- `Active: inactive (dead)` — it's installed but **not currently running**
- `disabled` — won't auto-start on boot (separate from whether it's running right now)

### "Unit could not be found" — a different situation entirely
```
Unit ntp.service could not be found.
```
This means the software **isn't installed at all** — there's nothing for systemd to manage. Different from "inactive," which means installed-but-off. Confirm with:
```bash
dpkg -l | grep packagename
which programname
```

### Service names don't always match the daemon name
Example: the actual program is `sshd`, but Kali's systemd unit is named `ssh.service`, not `sshd.service`. The daemon only actually launches once the service is *started* — `sshd` won't appear in `ps aux` until `systemctl start ssh` has been run.

## Why this all matters for security

- **Attack surface = running services.** Every active network-listening service is a potential entry point. Hardening often means disabling services you don't actually need.
- **Persistence technique (attacker side):** a common way to survive a reboot after compromising a machine is creating a malicious daemon/service disguised with a legitimate-looking name.
- **Investigation (defender side):** `systemctl list-units --type=service` with an eye for anything unfamiliar is a real step in auditing a system — an unexpected service name is worth investigating.
- Related daemons worth knowing exist: `cron` (scheduled task runner), `systemd-timesyncd` (lightweight time sync, often replacing the older standalone `ntp` package on modern Debian/Kali systems).
