# File & Directory Management

## Creating

| Command | Purpose |
|---|---|
| `mkdir name` | Create a new directory |
| `mkdir -p a/b/c` | Create nested directories in one go |
| `touch file.txt` | Create a new, empty file |

## Deleting

| Command | Purpose |
|---|---|
| `rmdir name` | Remove an **empty** directory only — refuses if it has contents (a safety feature) |
| `rm file.txt` | Delete a single file |
| `rm -r folder/` | Delete a folder and everything inside it (recursive) |
| `rm -rf folder/` | Same, but forced — no confirmation prompts |

**`rm -rf` has no undo.** Always run `pwd` and `ls` first to confirm exactly where you are and what you're about to delete before running it. One real mistake from my own notes: deleting a user with `userdel` (without `-r`) leaves their home directory behind as an orphaned folder — confirmed by checking `/home` afterward and seeing it still listed.

## Moving, copying, renaming

| Command | Purpose |
|---|---|
| `mv file.txt /path/` | Move a file (removes it from the original location) |
| `mv old.txt new.txt` | Rename a file (same folder = rename, not move) |
| `cp file.txt /path/` | Copy a file (keeps the original in place) |
| `cp -r folder/ /path/` | Copy a directory recursively |

**Mental shortcut:** `mv` = "move it" (one copy exists after), `cp` = "copy it" (two copies exist after).

## Editing files with nano

```bash
nano filename.txt
```

- Opens the file for editing (creates it if it doesn't exist)
- `Ctrl + O` — save (you'll be asked to confirm the filename)
- `Ctrl + X` — exit
- No autosave — always save before exiting

Other editors that exist: `vim`, `emacs` — different shortcut systems entirely, worth knowing they exist even if nano is the easiest starting point.

## Writing to files without an editor: redirection

Different from piping (`|`, which sends output to another **command**) — redirection (`>`, `>>`) sends output to a **file**.

| Operator | Behavior |
|---|---|
| `command > file.txt` | **Overwrites** the file completely with the command's output |
| `command >> file.txt` | **Appends** the output to the end of the file, keeping existing content |

```bash
echo "first line" > notes.txt     # notes.txt now contains just this line
echo "second line" >> notes.txt   # second line added, first line preserved
echo "wipe it" > notes.txt        # notes.txt now contains ONLY "wipe it"
```

Both `>` and `>>` create the file if it doesn't already exist.

**Why this matters for security work:** saving command output for later analysis/reporting instead of losing it when the terminal scrolls:
```bash
nmap -sV 192.168.1.1 > scan_results.txt
cat /var/log/auth.log | grep "Failed password" >> suspicious_logins.txt
```

**Safety habit:** default to `>>` unless you specifically want to wipe a file — `>` has caused real accidental data loss for people who meant to append.

## Checking file types

```bash
file somefile      # identifies what kind of file it actually is
```
Useful since file extensions in Linux are just naming convention, not enforced — `file` tells you what something actually is at the byte level.
