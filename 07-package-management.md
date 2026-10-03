# Package Management on Kali (Debian-based)

## apt vs dpkg — what each actually does

| Tool | Role |
|---|---|
| `dpkg` | Lower-level — actually installs a `.deb` file. Does NOT resolve dependencies automatically. |
| `apt` | Higher-level, built on `dpkg` — manages official repositories, auto-resolves dependencies, handles updates |

**Rule of thumb:**
- Downloaded a standalone `.deb` from a website (e.g., Discord)? → `sudo dpkg -i file.deb`
- Installing something from Kali's own repos (most security tools)? → `sudo apt install toolname`
- `dpkg -i` complains about missing dependencies? → `sudo apt --fix-broken install`

## Everyday apt commands

```bash
sudo apt update                      # refresh the list of available package versions (installs nothing yet)
sudo apt upgrade                     # install newer versions, but won't remove packages to resolve conflicts
sudo apt full-upgrade                # same, but WILL remove/replace packages if needed — recommended for Kali specifically, since it's a rolling release
sudo apt install packagename         # install a specific package
sudo apt remove packagename          # uninstall, but leave config files behind
sudo apt purge packagename           # uninstall AND remove config files
sudo apt search keyword              # search repos by keyword
sudo apt show packagename            # detailed info on a package before installing
apt list --installed                 # list everything currently installed
apt list --installed | grep ^name    # check if a specific package is installed (^ = start of line, avoids partial-name false matches)
```

**Good habit:** run `sudo apt update && sudo apt full-upgrade -y` weekly. Kali is a rolling release — small, frequent updates keep the system consistent and prevent slow, painful catch-up upgrades later.

### Troubleshooting apt

```bash
sudo apt full-upgrade --fix-missing          # skip packages that fail to download, continue with the rest
sudo apt -o Acquire::ForceIPv4=true update    # force IPv4 if hitting connection errors over IPv6
sudo apt clean                                # clear cached package files
```

A "keep current version?" prompt for a config file (e.g. `sudoers`) during upgrade: default to keeping your current version (`N`) unless you've deliberately customized it — if you have, use `D` to review the diff before deciding, never blindly overwrite `/etc/sudoers`.

## Installing software — the other methods

### Snap
```bash
sudo apt install snapd                       # Kali doesn't include this by default (Ubuntu does)
sudo systemctl enable --now snapd.socket
sudo systemctl enable --now snapd.apparmor   # needed for snap apps to actually launch
sudo snap install --classic packagename
```

### pip (Python packages)
```bash
pip3 install -r requirements.txt
```
Modern Kali blocks installing directly into the system Python ("externally-managed-environment" error) — use a virtual environment instead:
```bash
sudo apt install python3-full
python3 -m venv venv
source venv/bin/activate      # prompt shows (venv) when active
pip install -r requirements.txt
deactivate                    # when done
```
Inside an activated venv, `pip` and `pip3` point to the exact same isolated Python — no difference between them in that context.

**Why venvs matter:** each tool cloned from GitHub may need different, potentially conflicting package versions. Isolating each project's dependencies avoids breaking other tools or the system Python.

## Safety principle for any install method

The real risk factor isn't *which* installer you use (apt/dpkg/snap are all reasonably safe) — it's **the source and publisher**. Official repositories, official vendor websites, and verified publishers (shown with a ✓ in Snap, for example) are low-risk. Random unofficial download sites or unverified scripts are where actual risk lives, regardless of package format.

## File types you'll encounter

| Extension | What it is |
|---|---|
| `.deb` | Native Debian/Kali package — install with `dpkg -i` |
| `.AppImage` | Portable app, no install needed — `chmod +x` then run directly |
| `.tar.gz` / `.tar.xz` | Compressed archive, often source code — extract with `tar -xzf` / `tar -xJf` |
| `.sh` | Shell script — read before running, `chmod +x` then `./script.sh` |
| `.py` | Python script — run with `python3 script.py` |
| `.exe` | Windows executable — won't run natively on Linux |
| `.rpm` | Red Hat-family package — not compatible with Debian/Kali |

**Always read a script before running it, especially with `sudo`.** Running unknown code as root is one of the most common ways to compromise your own system.
