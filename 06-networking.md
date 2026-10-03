# Networking Basics on Linux

## Checking your network identity

```bash
ip a                     # all interfaces and their IP addresses (modern tool)
ifconfig                 # older equivalent, still works on Kali but considered legacy
ip route                 # routing table — which gateway traffic uses
hostname                 # this machine's network name
```

### Reading `ip a` output

```
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> ...
    link/ether 08:00:27:8a:35:d2 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic noprefixroute eth0
```
- `link/ether ...` — the MAC address. VirtualBox adapters start with `08:00:27` — a useful forensic tell that a machine is a VirtualBox VM.
- `inet 10.0.2.15/24` — current IP address and subnet (CIDR notation; `/24` = addresses `10.0.2.0`–`10.0.2.255`)
- `dynamic` — IP assigned via DHCP, not manually set
- `10.0.2.x` is VirtualBox's default **NAT** network range — isolated from the outside world, only reachable through the host.
- `lo` — the loopback interface, always `127.0.0.1`, used for a machine to talk to itself.

## NetworkManager — what's actually managing your connection

Modern Kali uses NetworkManager, not the old-style `/etc/network/interfaces` file (which on a modern install usually only defines `lo`).

```bash
nmcli device status          # summary of each network device and its connection state
nmcli connection show        # saved connection profiles
systemctl status NetworkManager   # is the managing service itself running?
```

A useful "network health check" sequence:
```bash
systemctl status NetworkManager   # is the service alive?
nmcli device status               # is the connection active?
ip a                              # what's the actual address?
```

## Listening ports / open services

```bash
ss -tulnp      # modern tool — Tcp, Udp, Listening, Numeric (no name resolution), show Process
netstat -tulnp # older equivalent
```

Flags are stackable single-letter options (`-t -u -l -n -p` combined = `-tulnp`) — verify any flag combo you're unsure about with `command --help` or `man command`.

### Reading `ss -tulnp` output
```
Netid  State    Local Address:Port    Peer Address:Port
tcp    LISTEN   127.0.0.1:39025       0.0.0.0:*
```
- **`127.0.0.1:PORT`** — only accessible from the same machine (private, safe)
- **`0.0.0.0:PORT`** — accessible from any network interface, including external ones (worth scrutinizing if unexpected)

**Why this matters:** checking listening ports is a core step in both offense (what's attackable) and defense (what shouldn't be open) — an unexpected service listening on `0.0.0.0` is a real red flag worth investigating.

## IPv4 vs IPv6 connection issues

VirtualBox's default NAT networking sometimes has incomplete IPv6 routing even though an IPv6 address is assigned locally. If something (like `apt`) fails to connect over IPv6 specifically:

```bash
sudo apt -o Acquire::ForceIPv4=true update
```

Make it permanent:
```bash
echo 'Acquire::ForceIPv4 "true";' | sudo tee /etc/apt/apt.conf.d/99force-ipv4
```

For actually working with/learning IPv6 properly later, switching VirtualBox's network mode from **NAT** to **Bridged Adapter** gives the VM a real address directly on the host network (trade-off: less isolated, more exposed to the local network).

## SSH basics

- `sshd` — the actual SSH daemon program
- `ssh.service` — the systemd unit name on Debian/Kali that manages `sshd` (note: NOT `sshd.service` — a naming quirk worth remembering)

```bash
sudo systemctl status ssh     # check if it's running
sudo systemctl start ssh      # start it
sudo systemctl enable ssh     # auto-start on boot
```

Config file: `/etc/ssh/sshd_config` — edited with `nano`/`vim`, then requires a restart to take effect:
```bash
sudo nano /etc/ssh/sshd_config
sudo systemctl restart ssh
```

Important hardening settings in that file:
- `PermitRootLogin no` — blocks direct root login over SSH
- `PasswordAuthentication no` — forces key-based auth instead of passwords (much harder to brute-force)

Connecting to a remote machine as a client:
```bash
ssh username@ip_address
```

**Safety note:** enabling SSH on a VirtualBox NAT-mode VM is low-risk — it's not directly exposed to the internet, only reachable from the host/other local VMs.

## Why subdomain/network reconnaissance matters

Tools like `subfinder`/`amass` map out an organization's full footprint (subdomains, exposed services) — important because forgotten, less-secured assets (old dev environments, abandoned subdomains) are frequently the actual entry point attackers use, not the well-guarded main site. This applies both offensively (bug bounty, pentesting) and defensively (attack surface management — knowing what your own org has exposed).

**Passive enumeration** (querying public sources like search engines, certificate logs, DNS aggregators) is generally safe/legal even against large companies, since no traffic is sent to their actual servers. **Active scanning** (port scanning, connecting directly) requires explicit permission.
