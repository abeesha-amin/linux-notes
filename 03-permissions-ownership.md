# Permissions, Ownership & sudo

## Reading permission strings

```bash
ls -la
```
```
drwx------  5 root root 4096 Aug 27 10:00 .
-rw-r--r--  1 root root  571 Jan  1  2024 .bashrc
-rw-------  1 root root   20 Aug 27 10:00 .bash_history
```

Breaking down `-rw-r--r--`:
```
-  rw-  r--  r--
│   │    │    │
│   │    │    └── others: read only
│   │    └──────── group: read only
│   └───────────── owner: read + write
└───────────────── file type: - = regular file, d = directory
```

- **Owner** permissions — what the file's owning user can do
- **Group** permissions — what members of the owning group can do
- **Others** permissions — what everyone else can do
- `r` = read, `w` = write, `x` = execute, `-` = no permission

`drwx------` (like `/root`) means only the owner has any access — this is exactly why a normal user gets "permission denied" trying to `cd` into another user's home directory or `/root`.

## Changing permissions: chmod

```bash
chmod 700 file.txt     # owner: rwx, group: none, others: none
chmod 755 script.sh    # owner: rwx, group: r-x, others: r-x
chmod +x script.sh     # add execute permission for everyone
```

Numeric shorthand: `r=4, w=2, x=1`, added together per category (owner/group/others). `7 = rwx`, `5 = r-x`, `0 = none`.

## Changing ownership: chown

```bash
sudo chown username file.txt           # change user owner
sudo chown :groupname file.txt         # change group owner (note the colon)
sudo chown username:groupname file.txt # change both at once
```

`chmod` controls **what level of access** exists; `chown` controls **who** that access belongs to. They work together.

## sudo — temporary elevated privileges

- `sudo` = "superuser do" — temporarily runs a single command as root, asks for your password
- Only users listed in the **sudoers file** can use it
- Edit the sudoers file safely with `sudo visudo` — never edit `/etc/sudoers` directly with a plain text editor; `visudo` checks syntax before saving, preventing a broken file that could lock out sudo entirely

**Why not just log in as root permanently?** Running everything as root removes all safety guardrails — one typo in a destructive command does far more damage, there's no record of which specific user ran what, and a compromised root session is catastrophic. `sudo` keeps actions tied to your specific account and limits the blast radius of a mistake or compromise.

### Becoming root properly

| Command | Effect |
|---|---|
| `sudo command` | Run one command as root, then return to normal user |
| `sudo -i` | Full root login shell, loads root's own environment/home |
| `sudo -s` | Root shell, keeps your current environment |
| `exit` or `Ctrl+D` | Leave a root shell, return to normal user |

**Important gotcha:** `sudo cd /root` does NOT work. `cd` is a shell builtin, not a standalone program file — `sudo` can only run actual executable files, so it fails with "cd: command not found." Confirm this yourself: `type cd` shows `cd is a shell builtin`. The fix is `sudo -i` (or `sudo -s`) first, then `cd` normally inside that root shell.

**Good habit:** don't stay in a root shell longer than needed — do the specific task, then `exit` immediately. This follows the principle of least privilege.

## Quick checks

```bash
whoami    # which user am I right now?
id        # my UID, GID, and every group I belong to
```

`id` is genuinely useful for privilege auditing — e.g., discovering you're in the `sudo`, `docker`, or `wireshark` groups tells you a lot about what that account can do, which matters a lot during privilege escalation assessment.
