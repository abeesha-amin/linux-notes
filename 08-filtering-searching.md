# Filtering & Searching: grep, find, piping

## Piping — connecting commands together

```bash
command1 | command2
```
Sends the **output** of one command as the **input** to the next. Different from redirection (`>`/`>>`, covered in [02-file-management.md](./02-file-management.md)), which sends output to a **file** instead of another command.

```bash
ps aux | grep firefox          # list all processes, filter for ones mentioning "firefox"
apt list | less                # long output, scroll through comfortably
ls /home/analyst/reports | grep users   # filenames in a directory containing "users"
```

## grep — searching for text patterns

```bash
grep "pattern" file.txt              # lines in file.txt containing "pattern"
grep -i "pattern" file.txt           # case-insensitive
grep "pattern" file.txt -r            # recursive, search through a whole directory
```

Common real use: searching your own shell history for a command you used before:
```bash
grep "nmap" ~/.zsh_history
```

## Avoiding grep matching its own process

```bash
ps aux | grep sublime        # might match grep's own command line too
ps aux | grep [s]ublime      # bracket trick — grep now searches for a pattern, not literal text, so it stops matching itself
```

## find — searching the filesystem by criteria

```bash
find /starting/path -criteria value
```

The first argument is **where** to start searching; everything after is the **filter**. Without a filter, it returns everything under that path.

### By size
```bash
find / -size +100M              # files larger than 100MB, system-wide
find / -size +100M 2>/dev/null  # same, but hide permission-denied noise
sudo find / -size +100M 2>/dev/null   # as root, so no folders are skipped due to permissions
```
`2>/dev/null` redirects stderr (error messages — stream `2`) to `/dev/null`, Linux's "discard" file — hides errors without stopping the actual search.

### By name
```bash
find /home/kali -name "*log*"     # case-sensitive
find /home/kali -iname "*log*"    # case-insensitive
```
`*` = wildcard for zero or more unknown characters.

### By modification time
```bash
find /home/kali -mtime -3     # modified within the last 3 days
find /home/kali -mtime +7     # modified more than 7 days ago
find /home/kali -mmin -30     # modified within the last 30 minutes (minute-level precision)
```

### Combining criteria
```bash
find /var/log -iname "*error*" -mtime -7
```

## Finding a command when you don't know its name

```bash
apropos keyword        # search command descriptions by topic/keyword
man -k keyword          # identical to apropos, different invocation
whatis commandname      # one-line summary of a known command
man commandname          # full manual
commandname --help       # quick summary of flags, usually shorter than man
```
If `apropos` returns "nothing appropriate" on a fresh system, rebuild its search database: `sudo mandb`.

**Workflow for figuring out an unfamiliar task:** identify the core action verb (search? filter? list?) rather than just the noun, then `apropos` that verb — much faster than guessing command names or always reaching for a search engine.

## Why this matters for security work

- `find -mtime` is a real incident-response technique — recently modified files often reveal a freshly planted backdoor or recently altered config, standing out against a system's normal baseline.
- `grep` + piping is the backbone of log analysis — filtering huge log files down to just the relevant lines (e.g., `grep "Failed password" /var/log/auth.log`) is daily-driver analyst work.
- `/tmp` and world-writable directories are common `find` targets when hunting for anything an attacker might have dropped.
