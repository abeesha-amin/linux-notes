# User & Group Management

## Creating users

| Command | Behavior |
|---|---|
| `sudo adduser name` | Friendlier, interactive — prompts for password, creates home dir, sets shell to `/bin/bash` automatically |
| `sudo useradd name` | Lower-level, minimal — does NOT set a password or create a home dir by default |
| `sudo useradd -m name` | Same as above, but `-m` explicitly creates the home directory |
| `sudo useradd -m -s /bin/bash name` | Also explicitly sets the shell (otherwise defaults to whatever `/etc/default/useradd` specifies — often `/bin/sh` on Debian/Kali) |

Since `useradd` doesn't set a password, follow it with:
```bash
sudo passwd newusername
```

**Why `useradd` defaults to `sh` while `adduser` defaults to `bash`:** these are two separate tools with separate config files (`/etc/default/useradd` vs `/etc/adduser.conf`), each with their own default shell setting. Check either with:
```bash
cat /etc/default/useradd
cat /etc/adduser.conf | grep -i shell
```

## Modifying users

```bash
sudo usermod --shell /bin/bash username    # change login shell
sudo usermod -l newname oldname            # rename a user (-l = login name)
sudo usermod -d /home/newpath username     # change home directory
sudo usermod -L username                   # lock account (prevent login, keep everything else)
sudo usermod -U username                   # unlock account
```

### The -aG trap (important)

```bash
sudo usermod -aG groupname username   # ✅ correct — ADDS to the group, keeps existing groups
sudo usermod -G groupname username    # ❌ dangerous — REPLACES all existing group memberships
```
Forgetting `-a` is a classic, genuinely damaging mistake — it can silently strip a user out of `sudo` or other critical groups. Always use `-aG` together when adding a user to one more group.

## Deleting users

```bash
sudo userdel username        # deletes the account ONLY — home directory is left behind
sudo userdel -r username     # deletes the account AND the home directory
```

**Real lesson learned:** running `userdel` without `-r` leaves an orphaned home folder. If you try to recreate the same username afterward with the same home directory, it'll fail because that folder already exists. Either clean up manually (`sudo rm -rf /home/username`) or always use `-r` from the start if you're sure you want the data gone.

**Alternative to deleting:** `usermod -L` locks an account without destroying it — often the better real-world choice (e.g., when an employee leaves), since it preserves files/ownership for investigation or reassignment rather than destroying them outright.

## Groups

```bash
sudo groupadd groupname              # create a new group
cat /etc/group                       # list all groups and their members
sudo gpasswd -d username groupname   # remove a user from a specific group
```

## Checking who exists / who's in what

```bash
id username                  # UID, GID, and all group memberships for a specific user
cat /etc/passwd               # full list of all accounts on the system (including system accounts)
cut -d: -f1 /etc/passwd        # just usernames, cleanly
awk -F: '$3 >= 1000 {print $1}' /etc/passwd   # only real human users (UID 1000+), not system accounts
who                            # who's currently logged in
```

## Why this matters for security

- `/etc/passwd` and group membership are among the first things checked during both offensive enumeration and defensive auditing — an unexpected account or unexpected group membership is a real red flag.
- `/etc/shadow` holds password hashes (root-only readable) — a major target if an attacker gains root, since offline password cracking becomes possible once this is exfiltrated.
- Renamed accounts (`usermod -l`) are a known technique for disguising a malicious account under an innocent-looking name.
