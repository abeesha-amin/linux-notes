# Navigation & the Linux Filesystem

## The core navigation loop

Three commands you use constantly, together:

```bash
pwd          # "where am I?" — prints working directory
ls -la       # "what's here?" — lists files, including hidden ones
cd folder/   # "let me go look" — moves into a directory
```

| Command | Purpose |
|---|---|
| `pwd` | Print working directory — shows your current absolute path |
| `ls` | List files/folders in current directory |
| `ls -l` | Detailed list: permissions, owner, size, date |
| `ls -a` | Show hidden files too (anything starting with `.`) |
| `ls -la` | Both combined — the one you'll use most |
| `cd folder` | Move into a subfolder |
| `cd ..` | Move up one level |
| `cd ~` or `cd` | Jump to home directory |
| `cd /` | Jump to filesystem root |
| `cd -` | Go back to the previous directory |

## Absolute vs relative paths

- **Absolute path** — full path starting from root (`/home/kali/Documents`)
- **Relative path** — starts from wherever you currently are (`../projects`)
- `.` = current directory
- `..` = parent directory (one level up)
- `~` = shorthand for your home directory (`/home/kali` for user `kali`)

## The Filesystem Hierarchy Standard (FHS)

Everything branches from `/` (root). Key directories:

| Directory | Contains |
|---|---|
| `/bin`, `/usr/bin` | Essential command binaries (`ls`, `cat`, `cp` — literally live here as files) |
| `/sbin`, `/usr/sbin` | System admin binaries, usually need root |
| `/etc` | System-wide configuration files — a major target for both attackers and defenders |
| `/home` | Personal folders per user (`/home/kali`) |
| `/root` | Home directory for the root user specifically — separated from `/home` for security |
| `/var` | Variable data — most importantly `/var/log`, where system logs live |
| `/tmp` | Temporary files, cleared on reboot. **Security note:** writable by everyone by default, so it's a common place attackers drop malicious files |
| `/dev` | Hardware devices represented as files |
| `/proc`, `/sys` | Virtual windows into the running kernel/processes in real time |
| `/mnt`, `/media` | Where external drives/shared folders get mounted |
| `/lib`, `/lib32`, `/lib64` | Shared library files programs depend on |
| `/opt` | Optional/third-party installed software |

Check it yourself: `man hier` gives the full official breakdown.

## Why commands are just files

Commands like `ls` aren't magic — they're actual executable files sitting in `/bin` or `/usr/bin`.

```bash
which ls        # shows the file path (or an alias, see note below)
file /usr/bin/ls  # confirms it's an executable binary
echo $PATH      # shows the list of folders Linux searches for commands
```

**Note on `which` and aliases:** Kali's zsh shell often has `ls` aliased (e.g., `ls --color=auto`). `which ls` may show the alias instead of the file path. Use `type -a ls` to see both the alias and the real underlying file.

**Security relevance:** Attackers sometimes replace legitimate command files with malicious versions (rootkits), or manipulate `$PATH` so a malicious file runs instead of the real command. Understanding "commands are just files, found via PATH" is the foundation for recognizing this kind of tampering.

## Permissions blocking navigation

If `cd` into another user's home folder (or `/root`) gets denied, it's the permission system working as intended — not a bug.

```bash
ls -la /home
```
```
drwxr-xr-x  5 kali    kali    4096 ... kali
drwx------  5 newuser newuser 4096 ... newuser
```

`drwx------` means only the owner has any access at all. See [03-permissions-ownership.md](./03-permissions-ownership.md) for the full breakdown of reading these strings.

## Reading file content

| Command | Purpose |
|---|---|
| `cat file` | Print the whole file to screen |
| `less file` | View file one page at a time (`space`=forward, `b`=back, `g`=top, `G`=bottom, `/word`=search, `q`=quit) |
| `head file` | First 10 lines (use `-n 5` for a custom count) |
| `tail file` | Last 10 lines (use `-n 20` for a custom count) |
| `tail -f file` | Watch a file live as new lines are added — essential for watching logs in real time |

**Security relevance:** `tail -f /var/log/auth.log` is a genuinely common way analysts monitor live login activity during an investigation.

## Good habits

- `ls -la`, not just `ls` — plain `ls` hides dotfiles, and hidden files are sometimes exactly where something important (or malicious) is placed.
- Run `pwd` often, especially before any destructive command (`rm -rf`, etc.) — confirm where you are before you act.
