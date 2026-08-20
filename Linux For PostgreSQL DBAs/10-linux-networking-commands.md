# Chapter 10: Linux Networking Commands

Linux provides powerful networking commands to configure, troubleshoot, and diagnose network
connections.

## 1. Network Configuration

| Command | Purpose | Example | Description |
|---|---|---|---|
| `ip a` / `ip addr` | Show all network interfaces and IPs | `ip a` | Displays all interfaces with IP addresses |
| `ip link` | Show network interfaces and status | `ip link` | Shows interfaces (UP/DOWN state) |
| `ifconfig` | Configure/display network interfaces (legacy) | `ifconfig` | Older tool, use `ip` instead |
| `hostname` | Show or set system hostname | `hostname` | Displays current hostname |
| `hostnamectl` | Set/change hostname permanently | `hostnamectl set-hostname myserver` | Changes hostname system-wide |

## 2. Check Connectivity

| Command | Purpose | Example | Description |
|---|---|---|---|
| `ping` | Check connectivity to a host | `ping google.com` | Sends ICMP echo requests |
| `ping -c 4 host` | Send a specific number of packets | `ping -c 4 google.com` | `-c` = count |
| `traceroute` | Trace the route packets take to a host | `traceroute google.com` | Shows each hop (each router) |
| `mtr` | Network diagnostic tool (ping + traceroute) | `mtr google.com` | Continuous network analysis |
| `tracepath` | Discover MTU and path to host | `tracepath google.com` | Like traceroute but MTU-aware |

## 3. DNS & Name Resolution

| Command | Purpose | Example | Description |
|---|---|---|---|
| `nslookup` | Query DNS to get domain or IP info | `nslookup google.com` | Interactive DNS lookup |
| `dig` | DNS lookup (more powerful) | `dig google.com` | Detailed DNS information |
| `host` | Simple DNS lookup | `host google.com` | Displays DNS record |
| `getent hosts` | Resolve hostnames (using NSS) | `getent hosts google.com` | Shows resolved IP using system config |

## 4. Network Statistics & Connections

| Command | Purpose | Example | Description |
|---|---|---|---|
| `ss` | Show sockets and connections | `ss -tuln` | Modern `netstat` replacement |
| `netstat` | Display network connections, routing tables | `netstat -tuln` | Older tool (install if needed) |
| `ip route` | Show or manage routing table | `ip route` | Displays routing table |
| `route -n` | Show routing table (legacy) | `route -n` | Older tool, less detail |
| `watch -n 1 ss -tuln` | Monitor connections in real time | `watch -n 1 ss -tuln` | Refresh output every 1 sec |

## 5. Port & Service Checking

| Command | Purpose | Example | Description |
|---|---|---|---|
| `nmap` | Network scanner tool | `nmap 192.168.1.1` | Scan host for open ports |
| `nmap -p 1-1000` | Scan specific port range | `nmap -p 1-1000 192.168.1.1` | Scan ports 1 to 1000 |
| `telnet` | Test connection to a port | `telnet google.com 80` | Check if a port is open (e.g., 80) |
| `nc` (netcat) | Read/write data across networks | `nc -zv google.com 80` | Check port connectivity |
| `lsof -i` | List open network files/sockets | `lsof -i :80` | Shows process using port 80 |

## 6. Download & File Transfer

| Command | Purpose | Example | Description |
|---|---|---|---|
| `wget` | Download files from web | `wget https://example.com/file.zip` | Non-interactive download |
| `curl` | Transfer data from or to server | `curl -O https://example.com/file.zip` | Supports many protocols |
| `scp` | Secure copy (file transfer) | `scp file.txt user@host:/path/` | Copy files to remote securely |
| `sftp` | Secure File Transfer (FTP over SSH) | `sftp user@host` | Interactive secure file transfer |

## Extra useful commands

- `ip neigh` — show ARP table (IP–MAC mapping)
- `ethtool eth0` — show/modify network interface settings
- `iwconfig` — show wireless interface configuration
- `iw dev` — show wireless devices and info
- `nmcli` — NetworkManager command line tool
- `systemctl status NetworkManager` — check NetworkManager status

### Tips

- Use `ip` command (modern and powerful) instead of the legacy `ifconfig`.
- Use `ss` instead of `netstat`.
- Root privileges (`sudo`) may be required for some commands.
- Install missing tools: `sudo apt install nmap dnsutils net-tools`

---

## Where a PostgreSQL DBA uses this

Almost every "can't connect to the database" ticket is really a networking ticket in
disguise, and this is the toolbox for diagnosing it layer by layer.

### Is PostgreSQL even listening?

```bash
ss -tuln | grep 5432          # is anything listening on port 5432?
sudo lsof -i :5432             # which process owns port 5432?
```
If nothing is listening, check `listen_addresses` in `postgresql.conf` — it may be bound to
`localhost` only when you need `0.0.0.0` (all interfaces) or a specific IP for remote access.

### "Connection refused" vs "timed out" — different diagnoses

```bash
telnet dbhost 5432    # "Connection refused" = port closed / PostgreSQL not listening on it
                        # "timed out" = firewall dropping the packet, or wrong network path
nc -zv dbhost 5432      # quick, scriptable equivalent of the telnet check
```
This distinction alone tells you whether to look at `pg_hba.conf`/`listen_addresses`
(refused) or at security groups/firewalls (timeout).

### Checking the actual client → server path

```bash
ping dbhost                    # is the host reachable at all?
traceroute dbhost               # where along the path does it break?
ip route                        # what's the outbound routing config on this box?
```

### Confirming which interfaces PostgreSQL should bind to

```bash
ip a                            # list this server's own IPs — match against listen_addresses
hostname -I                      # quick way to get this host's IP addresses
```

### Watching active connections to PostgreSQL in real time

```bash
ss -tn state established '( dport = :5432 or sport = :5432 )'
watch -n 1 "ss -tn | grep 5432"
```
This is a Linux-level cross-check against PostgreSQL's own `pg_stat_activity` — useful when
you suspect connections are piling up at the OS/network layer (e.g. a connection pooler
misbehaving) rather than inside PostgreSQL itself.

### Replication and streaming links

```bash
ping replica-host
nc -zv replica-host 5432
ss -tn | grep 5432 | grep <primary_ip>   # confirm a WAL streaming connection is actually up
```

### Moving backups and files between hosts

```bash
scp basebackup.tar.gz postgres@standby:/backups/
scp -r /backups/2026-08-16/ dr-host:/restore-staging/
```
`scp`/`sftp` are the standard tools for shipping `pg_basebackup` output or `pg_dump` files
between primary, standby, and offsite/DR hosts securely.

### DNS problems affecting connection strings

```bash
dig db-primary.internal
nslookup db-primary.internal
getent hosts db-primary.internal
```
Useful when an application's connection string uses a hostname and clients suddenly can't
resolve it — a very common failure mode after infrastructure or DNS changes, and one that
looks exactly like a database outage from the application's point of view.

## Hands-On Exercises — PostgreSQL DBA Bootcamp Style

This replays the very first thing done in the bootcamp — finding the lab VM's IP address to
even connect to it via PuTTY — then extends it to PostgreSQL connectivity.

1. **Find your own IP address** (the same information PuTTY needs to connect):
   ```bash
   ip a
   ```
   or the legacy command shown in the bootcamp:
   ```bash
   ifconfig
   ```
   Identify the IP bound to your primary network interface — this is what you'd type into
   PuTTY's "Host Name (or IP address)" field.

2. **Confirm PostgreSQL is listening**, once installed:
   ```bash
   ss -tuln | grep 5432
   ```
   If nothing appears, check `listen_addresses` in `postgresql.conf` (Chapter 7's `grep`
   exercise, applied here) — it may be restricted to `localhost` only.

3. **Test connectivity to your own instance as if you were a remote client:**
   ```bash
   telnet localhost 5432
   ```
   or
   ```bash
   nc -zv localhost 5432
   ```
   Then try it using the lab machine's actual IP from step 1 instead of `localhost`, to
   simulate a real remote client connection.

4. **Confirm the hostname resolves correctly**, matching the `lab01` style hostname pattern
   from the bootcamp prompt:
   ```bash
   hostname
   hostname -I
   ```

**Next:** [Chapter 11 — Package Management](11-linux-package-management.md)
**Previous:** [Chapter 9 — Disk & Memory Commands](09-linux-disk-memory-commands.md)
